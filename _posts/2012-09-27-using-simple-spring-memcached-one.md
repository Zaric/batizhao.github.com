---
layout: post
title: "使用 Simple-Spring-Memcached: SingleCache"
category: java 
tags: 
- spring
- memcached
- mybatis
- java
---
{% include JB/setup %}
上一篇大概讲了一下 SSM anotation。这章详细看一下 SingleCache 的使用。

## 首先是接下来的几个内容都会用到的两个 POJO

Role Model:

	public class Role implements Serializable {

	    private static final long serialVersionUID = -4708064835003250669L;
	
	    private Long id;
	
	    private String name;
	
	    @CacheKeyMethod
	    public String cacheKey() {
	        return id.toString();
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public void setId(Long id) {
	        this.id = id;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public boolean equals(Object obj) {
	        return EqualsBuilder.reflectionEquals(
	                this, obj);
	    }
	
	    public int hashCode() {
	        return HashCodeBuilder
	                .reflectionHashCode(this);
	    }
	
	    public String toString() {
	        return ToStringBuilder.reflectionToString(
	                this, ToStringStyle.MULTI_LINE_STYLE);
	    }
	}

User Model:

	public class User implements Serializable {

	    private static final long serialVersionUID = -822125371522084989L;
	
	    private Long id;
	
	    private String name;
	
	    private Role role;
	
	    @CacheKeyMethod
	    public String cacheKey() {
	        return id.toString();
	    }
	
	    public Long getId() {
	        return id;
	    }
	
	    public void setId(Long id) {
	        this.id = id;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public Role getRole() {
	        return role;
	    }
	
	    public void setRole(Role role) {
	        this.role = role;
	    }
	
	    public boolean equals(Object obj) {
	        return EqualsBuilder.reflectionEquals(
	                this, obj);
	    }
	
	    public int hashCode() {
	        return HashCodeBuilder
	                .reflectionHashCode(this);
	    }
	
	    public String toString() {
	        return ToStringBuilder.reflectionToString(
	                this, ToStringStyle.MULTI_LINE_STYLE);
	    }
	}

## 场景一：根据某个 user ID 查询某个 User。在更新时，更新缓存中的这个 User。

Method:

	@Override
    @ReadThroughSingleCache(namespace = "user", expiration = 600)
    public User getUser(@ParameterValueKeyProvider Long id) {
        return (User) sqlMapClientTemplate.queryForObject("getUser", id);
    }

    @Override
    @UpdateSingleCache(namespace = "user", expiration = 60)
    public void updateUser(@ParameterValueKeyProvider @ParameterDataUpdateContent User user) {
        sqlMapClientTemplate.update("updateUser", user);
    }
    
SQL:

	<update id="updateUser" parameterClass="user">
        UPDATE user
        SET name = #name#
        WHERE id = #id#
    </update>

    <select id="getUser" parameterClass="java.lang.Long" resultClass="user">
        SELECT * FROM user WHERE id = #id#
    </select>
    
只要有相同的 namespace

* 在 getUser 时，会根据 @ParameterValueKeyProvider 找到 User 对象的 @CacheKeyMethod 方法，到 Memcached 中 get user:id。
* 在 updateUser 时，会根据 @ParameterValueKeyProvider 找到 User 对象的 @CacheKeyMethod 方法，到 Memcached 中 set user:id    

## 场景二：根据某个 role ID 查询所有的 User。查询关联两个以上的 Model（User，Role），在更新  Role 时，需要让相关的缓存失效。

Method:

	@ReadThroughSingleCache(namespace = "user/list", expiration = 600)
    public List<User> getUsersByRoleId(@ParameterValueKeyProvider Long id) {
        return (List<User>) sqlMapClientTemplate.queryForList("getUsersByRoleId", id);
    }
    
SQL:

	<select id="getUsersByRoleId" parameterClass="java.lang.Long" resultClass="user">
		SELECT u.id, u.name, r.id as "role.id", r.name as "role.name"
	      FROM user u, user_role ur, role r
	     WHERE u.id = ur.userid and r.id = ur.roleid and ur.roleid = #id#
	</select>     
     
当更新 Role 时:

	@UpdateSingleCache(namespace = "role", expiration = 60)
    public void updateRole(@ParameterValueKeyProvider @ParameterDataUpdateContent Role role) {
        sqlMapClientTemplate.update("updateRole", role);
    } 
    
需要让 getUsersByRoleId 的缓存失效。这时最简单的办法是直接使用 annotation  @InvalidateSingleCache

	@UpdateSingleCache(namespace = "role", expiration = 60)
	@InvalidateSingleCache(namespace = "user/list")
    public void updateRole(@ParameterValueKeyProvider @ParameterDataUpdateContent Role role) {
        sqlMapClientTemplate.update("updateRole", role);
    }
    
但是，如果有多个类似的 Cache 需要清除，那这种办法就不适用了。这时可以每个 POJO 定义一个专门用来 invalidate 的类：

	@Component
	public class UserCache {

	    @InvalidateSingleCache(namespace = "user/list")
	    public void invalidateGetUsersByRoleId(@ParameterValueKeyProvider Long id){
	    }
	    
    ｝
    
在 Service 中调用相关的方法：

	@Service
	public class RoleManagerImpl implements RoleManager {

	    @Autowired
	    private RoleDao roleDao;
	
	    @Autowired
	    private UserCache userCache;
	
	    @Override
	    public void updateRole(Role role) {
	        roleDao.updateRole(role);
	        userCache.invalidateGetUsersByRoleId(role.getId());
	        groupCache.invalidate(role.getId());
	        ...
	    }
	} 

下一篇会讲一下 MultiCache 的使用。