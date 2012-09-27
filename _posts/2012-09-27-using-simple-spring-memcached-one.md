---
layout: post
title: "使用 Simple-Spring-Memcached（一）"
category: java 
tags: 
- spring
- memcached
- mybatis
- java
---
{% include JB/setup %}

场景：某个查询关联的两个以上的 Model，在更新其中一个时，需要让缓存失效。

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

