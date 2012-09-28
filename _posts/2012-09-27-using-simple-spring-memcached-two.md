---
layout: post
title: "使用 Simple-Spring-Memcached: MultiCache"
category: java 
tags: 
- spring
- memcached
- mybatis
- java
---
{% include JB/setup %}

上一篇讲了 SingleCache ，这篇主要讲一下 MultiCache 的使用。在此之前，要先理解一下 Namespace 和 Key 两个参数。在 Memcached，读写数据都是根据 namespace 和 key 来进行的。

## Namespace 和 Key

这里首先要理解 SSM 中的 namespace 这个参数。其实这个参数主要是和你的方法返回结果相关。在这篇和上一篇联系起来看，可以使用同一个 namespace。这里可以把 MultiCache 看成 SingleCache 的批量操作。因为无论是 SingleCache 还是 MultiCache ，最终的 Cache 操作其实就是

* get user:id

	{"v":{"me.batizhao.model.User":{"id":1000,"name":"Messi"}}}
	
* set user:id

	{"v":{"me.batizhao.model.User":{"id":1000,"name":"Messi"}}}


如果不理解 namespace 和 ParameterValueKeyProvider，会带来的问题是

* 缓存命中率（大大影响缓存使用效率）
* 内容空间（占用更多的内存）
* 需要更多的清除操作（程序复杂度增加）


## 场景一：根据某几个 user ID 查询所有的 User。在更新某个 user 时，同时更新相关的缓存

Method:

	@Override
    @ReadThroughMultiCache(namespace = "user", expiration = 60)
    public List<User> getUsersByUserIds(@ParameterValueKeyProvider List<Long> ids) {
        return (List<User>) sqlMapClientTemplate.queryForList("getUsersByUserIds", ids);
    }
    
SQL:

	<select id="getUsersByUserIds" parameterClass="list" resultClass="user">
        SELECT u.id, u.name
        FROM user u
        WHERE u.id in
        (<iterate conjunction=",">
            #ids[]#
        </iterate>)
    </select>
    
Memcached Log:

	<21 get user:1000 user:1001
	>21 sending key user:1000
	>21 END
	<21 set user:1001
	>21 STORED
	
更新。这里 Batch 调用 updateUser，因为方法没有返回值，所以只能使用 @ParameterDataUpdateContent

	@Override
    @UpdateMultiCache(namespace = "user", expiration = 60)
    public void updateUsersByUserIds(@ParameterValueKeyProvider @ParameterDataUpdateContent final List<User> users) {
        sqlMapClientTemplate.execute(new SqlMapClientCallback() {
            @Override
            public Object doInSqlMapClient(SqlMapExecutor executor) throws SQLException {
                executor.startBatch();

                for(User user: users){
                    executor.update("updateUser", user);
                }

                executor.executeBatch();

                return null;

            }
        });
    }	
	
看一下 Memcached Log，user:1000 直接从 Cache 返回，只有 user:1001 做了数据库操作。这是因为我在执行	getUsersByUserIds 之前，执行了上一篇中的 getUser(1000L)，先生成了 Cache。        

## 场景二：根据某几个 role ID 查询所有的 User。查询关联两个以上的 Model，在更新其中一个时，需要让相关的缓存失效

Method:

	@Override
    @ReadThroughMultiCache(namespace = "user/getUsersByRoleIds", expiration = 60)
    public List<User> getUsersByRoleIds(@ParameterValueKeyProvider final List<Long> ids) {
        return (List<User>) sqlMapClientTemplate.queryForList("getUsersByRoleIds", ids);
    }
    
*这里为什么不用 namespace = "user"？*   

SQL:

	<select id="getUsersByRoleIds" parameterClass="list" resultClass="user">
        SELECT u.id, u.name, r.id as "role.id", r.name as "role.name"
        FROM user u, user_role ur, role r
        WHERE u.id = ur.userid and r.id = ur.roleid and ur.roleid in
        (<iterate conjunction=",">
            #ids[]#
        </iterate>)
    </select>
    
看一下返回的结果集：

	+------+-------+---------+------------+
	| id   | name  | role.id | role.name  |
	+------+-------+---------+------------+
	| 1000 | Tom   |       1 | ROLE_ADMIN |
	| 1001 | Jerry |       2 | ROLE_USER  |
	| 1002 | Jack  |       1 | ROLE_ADMIN |
	+------+-------+---------+------------+ 
	
测试代码是这样的：

	@Test
    public void testGetUsersByRoleIds() throws Exception {
        List<Long> list = Arrays.asList(1L, 2L);

        List users = userDao.getUsersByRoleIds(list);

        log.info("users: " + users);

        assertNotNull(users);
        assertEquals(3, users.size());
    }	
	
这个方法不会生成任何缓存。SSM 会告诉你

	com.google.code.ssm.aop.ReadThroughMultiCacheAdvice: Did not receive a correlated amount of data from the target method.
	
因为 SSM 会先根据结果集生成缓存（使用返回对象 User 的 @CacheKeyMethod 方法）

	set user/getUsersByRoleIds:1 user/getUsersByRoleIds:2
	
但是返回的结果集中没有 1 和 2，就算有，也不是我们想要的 role.id，而是 user.id。刚开始我尝试和 SSM 的开发者沟通[这个问题](http://code.google.com/p/simple-spring-memcached/issues/detail?id=10)。他让我尝试修改一下 annotation

	@Override
    @ReadThroughMultiCache(namespace = "user/getUsersByRoleIds", expiration = 60, option = @ReadThroughMultiCacheOption(generateKeysFromResult = true))
    public List<User> getUsersByRoleIds(@ParameterValueKeyProvider final List<Long> ids) {
        return (List<User>) sqlMapClientTemplate.queryForList("getUsersByRoleIds", ids);
    }
    
之后变成这个样子：
	
	get user/getUsersByRoleIds:1 user/getUsersByRoleIds:2
	
	set user/getUsersByRoleIds:1000 8 60 120
	{"v":{"me.batizhao.model.User":{"id":1000,"name":"Tom","role":{"me.batizhao.model.Role":{"id":1,"name":"ROLE_ADMIN"}}}}}
	
	set user/getUsersByRoleIds:1001 8 60 121
	{"v":{"me.batizhao.model.User":{"id":1001,"name":"Jerry","role":{"me.batizhao.model.Role":{"id":2,"name":"ROLE_USER"}}}}}
	
	set user/getUsersByRoleIds:1002 8 60 121
	{"v":{"me.batizhao.model.User":{"id":1002,"name":"Jack","role":{"me.batizhao.model.Role":{"id":1,"name":"ROLE_ADMIN"}}}}}
	
在这个返回的 List 中，role.id 不可以做为 Key，因为不是唯一的。试了一下返回 HaspMap，@ReadThroughMultiCache 只支持 List。所以想到两种解决办法：

* 循环调用上一篇中的 getUsersByRoleId 方法。
* 使用 @ReadThroughAssignCache（缺点是缓存做为一整块，不能像 @ReadThroughMultiCache 一样对单个 User 做操作了），到时只能整体清除。

第一个方案可以自行试验一下，下一篇讲 AssignCache 如何解决这个问题。

	
	    	  