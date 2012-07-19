---
layout: post
title: "自定义 CAS assertion 返回值"
category: cas
tags: 
- cas
- ldap
---
{% include JB/setup %}

这篇主要是讲 CAS 和 LDAP 集成时，如何返回一些特别的内容到 Spring Security 客户端。这样我们可以从 LDAP 取到 user 的 role，
和 Spring Security 进行比对授权。但是，CAS 默认配置的 UsernamePasswordCredentialsToPrincipalResolver 不允许我们在与
Spring Security 集成时传递回特别的属性信息，所以我们需要对 CAS 进行修改允许我们这样做。

CAS 提供了高级的配置使客户端与 CAS 服务端进行数据交换。在 CAS 服务器传递 ticket 校验结果时，可以将基于 CAS 认证时查询到的信息进行传递。
这些信息以键值对的方式进行传递，并可以包含用户相关的任何数据。我们将会使用这个功能在 CAS 响应中传递用户的属性，包括 GrantedAuthority 信息。

## 1. deployerConfigContext.xml

CAS 服务器的 org.jasig.cas.authentication.AuthenticationManager 负责基于提供的凭证信息进行用户认证。
与 Spring Security 很相似，实际的认证委托给了一个（或更多）实现了 org.jasig.cas.authentication.handler.AuthenticationHandler 接口的处理类
（在 Spring Security 中对应的接口是 AuthenticationProvider ）。
 
org.jasig.cas.authentication.principal.CredentialsToPrincipalResolver 用来将传递进来的安全实体信息转换成完整的
org.jasig.cas.authentication.principal.Principal（类似于 Spring Security中 UserDetailsService 实现所作的那样）。

## 2. 默认返回值

HomeController.java

	@Controller
	@RequestMapping("home")
	public class HomeController {
	
		@RequestMapping(method = RequestMethod.GET)
		public ModelAndView home() {
			ModelAndView mv = new ModelAndView("home");
	
			final Authentication auth = SecurityContextHolder.getContext().getAuthentication();
			mv.addObject("auth", auth);
			if (auth instanceof CasAuthenticationToken) {
				mv.addObject("isCasAuthentication", Boolean.TRUE);
			}
	
			return mv;
		}
	}

home.jsp

	<h1>View Profile</h1>

	<p>
		Some information about you, from CAS:
	</p>
	<ul>
		<li><strong>Auth:</strong> ${auth}</li>
		<li><strong>Username:</strong> ${auth.principal}</li>
		<li><strong>Credentials:</strong> ${auth.credentials}</li>
		<c:if test="${isCasAuthentication}">
			<li><strong>Assertion:</strong> ${auth.assertion}</li>
			<li><strong>Assertion Attributes:</strong>
				<c:forEach items="${auth.assertion.attributes}" var="attr">
					${attr.key}:${attr.value}<br/>
				</c:forEach>
			</li>
			<li><strong>Assertion Attribute Principal:</strong> ${auth.assertion.principal}</li>
			<li><strong>Assertion Principal Attributes:</strong>
				<c:forEach items="${auth.assertion.principal.attributes}" var="attr">
					${attr.key}:${attr.value}<br/>
				</c:forEach>
			</li>
		</c:if>
	</ul>
	
这个 jsp 的返回结果大致是这样的，只可以返回 Username。

	Auth: org.springframework.security.cas.authentication.CasAuthenticationToken@1358c3fc: Principal: org.springframework.security.core.userdetails.User@aa9c3074: Username: zhangsan; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Granted Authorities: I'M ZHANGSAN.; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffed504: RemoteIpAddress: 127.0.0.1; SessionId: 748C80EE4C063D18F2461486F82FC2C8; Granted Authorities: I'M ZHANGSAN. Assertion: org.jasig.cas.client.validation.AssertionImpl@5b3ba312 Credentials (Service/Proxy Ticket): ST-3-zbmPK3BMjrAmTEwwMlxi-cas
	Username: org.springframework.security.core.userdetails.User@aa9c3074: Username: zhangsan; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Granted Authorities: I'M ZHANGSAN.
	Credentials: ST-3-zbmPK3BMjrAmTEwwMlxi-cas
	Assertion: org.jasig.cas.client.validation.AssertionImpl@5b3ba312
	Assertion Attributes:
	Assertion Attribute Principal: zhangsan
	Assertion Principal Attributes: 
	
## 3. 自定义返回值

我们需要先修改 deployerConfigContext.xml，重新定义 attributeRepository 这个 Bean

	<bean id="attributeRepository"
          class="org.jasig.services.persondir.support.ldap.LdapPersonAttributeDao">
        <property name="contextSource" ref="contextSource"/>
        <property name="requireAllQueryAttributes" value="true"/>
        <property name="baseDN" value="ou=people,dc=dev,dc=org"/>
        <property name="queryAttributeMapping">
            <map>
                <entry key="username" value="uid"/>
            </map>
        </property>
        <property name="resultAttributeMapping">
            <map>
                <entry key="uid" value="username"/>
                <entry key="displayName" value="displayName"/>
                <entry key="description" value="role"/>
                <entry key="telephoneNumber" value="telephoneNumber"/>
            </map>
        </property>
    </bean>
    
这个类将 Principal 与后端的 LDAP 目录进行匹配。queryAttributeMapping 属性将 Principal 的 username 域与 LDAP 的
uid 属性相匹配。resultAttributeMapping 中的键值对 key 表示 LDAP 中的属性，而 value 表示返回的 assertion 属性。
而这个 role  属性就是 GrantedAuthorityFromAssertionAttributesUserDetailsService 要进行查找的。
    
在客户端需要配置 GrantedAuthorityFromAssertionAttributesUserDetailsService ，它的工作就是读取 CAS assertion
、寻找特定的属性并将属性值与用户的 GrantedAuthority 进行匹配。假设 assertion 返回了一个名为 role 的属性。我们只需要在
applicationContext-security.xml 中简单配置一个新的 bean

	<bean id="authenticationUserDetailsService"
          class="org.springframework.security.cas.userdetails.GrantedAuthorityFromAssertionAttributesUserDetailsService">
        <constructor-arg>
            <array>
                <value>role</value>
            </array>
        </constructor-arg>
    </bean>
    
但现在还不行，我们还需要在 Server 端做一些工作。继续 deployerConfigContext.xml，找到 UsernamePasswordCredentialsToPrincipalResolver，增加
attributeRepository 属性

	<bean class="org.jasig.cas.authentication.principal.UsernamePasswordCredentialsToPrincipalResolver">
		<property name="attributeRepository" ref="attributeRepository"/>
	</bean>

找到 serviceRegistryDao，在所有的 list bean 中增加

	<property name="ignoreAttributes" value="true" />

编辑 WEB-INF/view/jsp/protocol/2.0/casServiceValidationSuccess.jsp，在 cas:user 后边增加以下内容

	<cas:attributes>
		<c:forEach var="attr"
				   items="${assertion.chainedAuthentications[fn:length(assertion.chainedAuthentications)-1].principal.attributes}"
				   varStatus="loopStatus" begin="0"
				   end="${fn:length(assertion.chainedAuthentications[fn:length(assertion.chainedAuthentications)-1].principal.attributes)-1}"
				   step="1">
			<cas:${fn:escapeXml(attr.key)}>${fn:escapeXml(attr.value)}</cas:${fn:escapeXml(attr.key)}>
		</c:forEach>
	</cas:attributes>
	
现在可以 mvn package ，重新部署 CAS Server，重新登录访问 home.jsp，可以看到结果多出了一些内容	
    
    Auth: org.springframework.security.cas.authentication.CasAuthenticationToken@1358c3fc: Principal: org.springframework.security.core.userdetails.User@aa9c3074: Username: zhangsan; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Granted Authorities: I'M ZHANGSAN.; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffed504: RemoteIpAddress: 127.0.0.1; SessionId: 748C80EE4C063D18F2461486F82FC2C8; Granted Authorities: I'M ZHANGSAN. Assertion: org.jasig.cas.client.validation.AssertionImpl@5b3ba312 Credentials (Service/Proxy Ticket): ST-3-zbmPK3BMjrAmTEwwMlxi-cas
	Username: org.springframework.security.core.userdetails.User@aa9c3074: Username: zhangsan; Password: [PROTECTED]; Enabled: true; AccountNonExpired: true; credentialsNonExpired: true; AccountNonLocked: true; Granted Authorities: I'M ZHANGSAN.
	Credentials: ST-3-zbmPK3BMjrAmTEwwMlxi-cas
	Assertion: org.jasig.cas.client.validation.AssertionImpl@5b3ba312
	Assertion Attributes:
	Assertion Attribute Principal: zhangsan
	Assertion Principal Attributes: username:zhangsan
	telephoneNumber:18918588940
	role:ROLE_USER.
	displayName:zhangsan
	
这样在客户端的 Spring Security 中就可以实现授权比对

	<sec:intercept-url pattern="/user" access="ROLE_USER"/>
	
## 4. 参考

* [高级CAS配置](http://sishuok.com/forum/blogPost/list/3981.html)

	


