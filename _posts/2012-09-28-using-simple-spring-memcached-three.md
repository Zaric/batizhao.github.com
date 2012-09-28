---
layout: post
title: "使用 Simple-Spring-Memcached: AssignCache"
category: java 
tags: 
- spring
- memcached
- mybatis
- java
---
{% include JB/setup %}

接上一篇的 MultiCache ，这篇主要讲一下 AssignCache 的使用。继续解决上一个场景中没有解决的问题。

## 根据某几个 role ID 查询所有的 User。查询关联两个以上的 Model，在更新其中一个时，需要让相关的缓存失效

Method:

	@Override
    @ReadThroughAssignCache(assignedKey = "user/getUsersByRoleIds", namespace = "user", expiration = 60)
    public List<User> getUsersByRoleIds(@ParameterValueKeyProvider final List<Long> ids) {
        return (List<User>) sqlMapClientTemplate.queryForList("getUsersByRoleIds", ids);
    }
    
SQL:

	<select id="getUsersByRoleIds" parameterClass="list" resultClass="user">
        SELECT u.id, u.name, r.id as "role.id", r.name as "role.name"
        FROM user u, user_role ur, role r
        WHERE u.id = ur.userid and r.id = ur.roleid and ur.roleid in
        (<iterate conjunction=",">
            #ids[]#
        </iterate>) order by r.id
    </select>
    
Unit Test:

	@Test
    public void testGetUsersByRoleIds() throws Exception {
        List<Long> list = Arrays.asList(1L, 2L);

        List users = userDao.getUsersByRoleIds(list);

        log.info("users: " + users);

        assertNotNull(users);
        assertEquals(3, users.size());
    }    
    
Log4j log:

	set user:user/getUsersByRoleIds 8 60 378
	
	{"v":{"java.util.ArrayList":[{"me.batizhao.model.User":{"id":1000,"name":"Tom","role":{"me.batizhao.model.Role":{"id":1,"name":"ROLE_ADMIN"}}}},{"me.batizhao.model.User":{"id":1002,"name":"Jack","role":{"me.batizhao.model.Role":{"id":1,"name":"ROLE_ADMIN"}}}},{"me.batizhao.model.User":{"id":1001,"name":"Jerry","role":{"me.batizhao.model.Role":{"id":2,"name":"ROLE_USER"}}}}]}}

这个场景不太适合做 @UpdateAssignCache，这里使用 [Simple-Spring-Memcached: SingleCache](/java/2012/09/27/using-simple-spring-memcached-one/) 中的 UserCache 类，增加一个方法

	@InvalidateAssignCache(assignedKey = "user/getUsersByRoleIds", namespace = "user")
    public void invalidategetUsersByRoleIds(){
    }
    
然后在 RoleManagerImpl.updateRole() 中增加一句

	userCache.invalidategetUsersByRoleIds();
	
先执行 DAO Unit Test:

	@Test
    public void testGetUsersByRoleIds() throws Exception {
        List<Long> list = Arrays.asList(1L, 2L);

        List users = userDao.getUsersByRoleIds(list);

        log.info("users: " + users);

        assertNotNull(users);
        assertEquals(3, users.size());
    }
    
Memcached Log:
    
    <21 get user:user/getUsersByRoleIds
	>21 END
	<21 set user:user/getUsersByRoleIds 8 60 378
	>21 STORED	
	
再执行 Service Unit Test:

	@Test
    public void testUpdateRole() throws Exception {
        Role role = new Role();
        role.setId(1L);
        role.setName("ROLE_ADMIN");

        roleManager.updateRole(role);

        role = roleManager.getRole(1L);

        log.info("Role: " + role);

        assertEquals("ROLE_ADMIN", role.getName());

    }
    	    	
Memcached Log:

	<21 delete user:user/getUsersByRoleIds
	>21 DELETED	        

以上章节的所有代码在 [Github](https://github.com/batizhao/spring-mybatis-memcached).