---
layout: post
title: "使用 Simple-Spring-Memcached"
category: java 
tags: 
- spring
- memcached
- mybatis
- java
---
{% include JB/setup %}

因为公司有的项目还运行在 MyBatis2 上边，并且 [SSM](http://code.google.com/p/simple-spring-memcached/) 暂时还不可以直接使用在 MyBatis3 上边（可以通过 Spring Cache 或者直接使用 [mybatis-memcached](http://www.mybatis.org/caches/memcached/) 来实现，但后一种方式不适合那种需要对 Cache 进行精细控制的场景）。所以这里主要写一下如何使用 SSM3 完成 Spring3 + Memcached + MyBatis2 的整合。

# 1. SSM Annotation

## SingleCache 类
操作单个 POJO 的 Cache 数据，由 ParameterValueKeyProvider 和CacheKeyMethod 来标识组装 key。

Java Code:
	
	@ReadThroughSingleCache(namespace = "user", expiration = 600)
	public User getUser(
	@ParameterValueKeyProvider Long id)
	
Memcache Log:

	get user:1
	set user:1
	
## MultiCache 类
操作 List 型的 Cache 数据，由 ParameterValueKeyProvider 和 CacheKeyMethod 来标识组装 key。

Java Code:

	@ReadThroughMultiCache(namespace = "user/getUsersByUserIds", expiration = 600)
    public List<User> getUsersByUserIds(@ParameterValueKeyProvider final List<Long> ids)
    
If ids=[1,2,3], Then Memcache Log:

	get user:1 user:2 user:3
	set user:1
	set user:2
	set user:3	    

## AssignCache 类
无参方法。指定 key 操作 Cache 数据，由 annotation 中的 assignedKey 指定 key。

Java Code:

	@ReadThroughAssignCache(assignedKey = "user/getAllUsers", namespace = "role", expiration = 600)
    public List<User> getAllUsers()

# 2. SingleCache Annotation
## @ReadThroughSingleCache
作用：生成 Cache。

过程：

* 在执行方法之前先检查缓存。
* 如果找到相应的 Key，返回 Value，方法 return。
* 如果没有找到相应的 Key，查询数据库，再 set cache，方法 return。

Key 生成规则：ParameterValueKeyProvider 指定的参数，如果该参数对象中包含 CacheKeyMethod 注解的方法，则调用其方法，否则调用 toString 方法。

## @InvalidateSingleCache
作用：清除 Cache 中的数据。

key 生成规则：

* 使用 ParameterValueKeyProvider 注解时，与 ReadThroughSingleCache 一致。
* 使用 ReturnValueKeyProvider 注解时，key 为返回的对象的 CacheKeyMethod 或 toString 方法生成。

Java Code:

	@InvalidateSingleCache(namespace = "user/list")
    public void invalidateGetUsersByRoleId(@ParameterValueKeyProvider Long id)        

## @UpdateSingleCache
作用：更新 Cache 中的数据。

Key 生成规则：ParameterValueKeyProvider 指定。

* ParameterDataUpdateContent：方法参数中的数据，作为更新缓存的数据。
* ReturnDataUpdateContent：方法调用后生成的数据，作为更新缓存的数据。
* 上述两个注解，必须与 Update* 系列的注解一起使用。

Java Code:

	@UpdateSingleCache(namespace = "user", expiration = 60)
    public void updateUser(@ParameterValueKeyProvider @ParameterDataUpdateContent User user)
    
    @UpdateSingleCache(namespace = "user", expiration = 60)
    @ReturnDataUpdateContent
    public void updateUser(@ParameterValueKeyProvider User user)

# 3. 使多个相关的 Cache 失效
Java Code:

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
    
但是，如果有多个 Cache 需要清除，那这种办法就不适用了。这时可以每个 POJO 定义一个专门用来 invalidate 的类：

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

参考：[使用SSM注解做缓存操作](http://www.colorfuldays.org/program/java/ssm_memcache/)
