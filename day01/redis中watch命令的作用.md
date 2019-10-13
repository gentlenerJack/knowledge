#redis 中watch 的命令的作用
一、redis 中的watch 的命令的作用

Redis Watch 命令用于监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断

二、redis 中的watch 的命令的用法

在MULTI命令执行之前，可以指定待监控的keys，然而在执行EXEC之前，如果被监控的keys发生修改，EXEC将放弃执行该事务队列中的所有指令。

一般在MULTI 命令前就用watch命令对某个key进行监控.如果想让key取消被监控，可以用unwatch命令

WHTCH是一个乐观锁，有利于减少并发中的冲突，提高吞吐量。

三、redis 中的watch 的命令的实现原理：

被监视的key会被保存在两个地方：

1、redisClient中的list * watched_keys中

typedef struct redisClient {

    // 被监视的键
    list *watched_keys;     /* Keys WATCHED for MULTI/EXEC CAS */

} redisClient;


所有被监视的key 都保存在redisClient中的list * watched_keys这个结构体中

typedef struct watchedKey {

// 被监视的key

    robj *key;

// 被监视的key所在的DB

    redisDb *db;

} watchedKey;


2、监视的key和所在的client组成一个dict

typedef struct redisDb {

    // 被监视的key 所在的dict.被监视的key和所在的client组成一个dict

    dict *watched_keys;         /* WATCHED keys for MULTI/EXEC CAS */



四、代码实现redis 中的watch 的命令：

1、业务逻辑：所有被监视的key 都保存在redisClient中的list * watched_keys这个结构体中，首先检测要监听的key是否已经存在redisClient中的watched_keys 这个list中，判断key相等的标准是key的字符串是否相等，key对应的数据的指针是否指向同一个数据，如果key对应的client 不存在的话，则新建一个clients指针，把key和client 作为dict 添加到watched_keys中

2、代码如下：

void watchCommand(redisClient *c) {

    int j;
 
    // 可以看到watch命令必须要在事务执行之前执行，REDIS_MULTI 这个falgs是在multi命令中置位的

	// 也就是watch函数必须在multi命令之前执行

    if (c->flags & REDIS_MULTI) {

        addReplyError(c,"WATCH inside MULTI is not allowed");

        return;

    }
 
    // 前面我们之前被监视的key是保存在redisClient中的list * watched_keys这个结构体中

    for (j = 1; j < c->argc; j++)

        watchForKey(c,c->argv[j]);
 
    addReply(c,shared.ok);

}
 
void watchForKey(redisClient *c, robj *key) {
 
    list *clients = NULL;

    listIter li;

    listNode *ln;

    watchedKey *wk;
 
    //首先检测key是否已经存在redisClient中的watched_keys 这个list中

    listRewind(c->watched_keys,&li);

    while((ln = listNext(&li))) {

        wk = listNodeValue(ln);

		// 判断相等的标准是key的字符串相等，db的指针指向同一个db

        if (wk->db == c->db && equalStringObjects(key,wk->key))

            return; /* Key already watched */

    }

    clients = dictFetchValue(c->db->watched_keys,key);

    //如果key对应的client 还不在的话，则通过listCreat新建一个clients指针，并

	// 把key和client 作为dict 添加到watched_keys中

	//clients 为null，说明这个client是第一次添加被监视的key

    if (!clients) { 

        // 值为链表

        clients = listCreate();

        // 关联键值对到字典

        dictAdd(c->db->watched_keys,key,clients);

        incrRefCount(key);

    }

    // 将客户端添加到链表的末尾

    listAddNodeTail(clients,c);
 
    // 将新的watchedkey 田间道wacthe_keys链表的结尾

    wk = zmalloc(sizeof(*wk));

    wk->key = key;

    wk->db = c->db;

    // key的引用计数加1

    incrRefCount(key);

    listAddNodeTail(c->watched_keys,wk);

}

仅仅是这个robj对应对应的refcount 即引用计数加1

void incrRefCount(robj *o) {

    o->refcount++;

}


五、与之相反的unwatch命令实现如下：

代码如下：

void unwatchAllKeys(redisClient *c) {

    listIter li;

    listNode *ln;
 
    //如果key没有被监视，就不必取消了，直接返回

    if (listLength(c->watched_keys) == 0) return;
 
    // 遍历watched_keys 中所有被监视的key

    listRewind(c->watched_keys,&li);

    while((ln = listNext(&li))) {

        list *clients;

        watchedKey *wk;
 
        /* Lookup the watched key -> clients list and remove the client

         * from the list */

        // 将client 从 被监视的ky中移除

        wk = listNodeValue(ln);

        // 取出客户端链表

        clients = dictFetchValue(wk->db->watched_keys, wk->key);

		// 断言client 不能为null

        redisAssertWithInfo(c,NULL,clients != NULL);
        
        // 删除client

        listDelNode(clients,listSearchKey(clients,c));
 
        /* Kill the entry at all if this was the only client */

        // client为null，则说明没有这个key 没有对应的clint有监视了，因此删除这个key

        if (listLength(clients) == 0)

            dictDelete(wk->db->watched_keys, wk->key);
 
        /* Remove this watched key from the client->watched list */

        // 移除key这个节点

        listDelNode(c->watched_keys,ln);

		// keys的引用计数减1

        decrRefCount(wk->key);

		//释放wk 占用的内存

        zfree(wk);

    }


六、redis中WATCH命令实现效果展示：

WATCH命令可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行。监控一直持续到EXEC命令（事务中的命令是在EXEC之后才执行的，所以在MULTI命令后可以修改WATCH监控的键值），如：

redis> SET key 1

OK

redis> WATCH key

OK

redis> SET key 2

OK

redis> MULTI

OK

redis> SET key 3

QUEUED

redis> EXEC

(nil)

redis> GET key

"2"

上例中在执行WATCH命令后、事务执行前修改了key的值（即SET key 2），所以最后事务中的命令SET key 3没有执行

七、redis中watch命令的应用场景：

在一个事务中只有当所有命令都依次执行完后才能得到每个结果的返回值，可是有的时候需要先获得一条命令的返回值，然后再根据这个值执行下一条命令，可是因为事务中的每个命令的执行结果都是最后一起返回的，所以无法将前一条命令的结果作为下一条命令的参数，也就是在执行SET命令时无法获得GET命令的返回值，也就无法根据结果执行下一条命令。但是我们可以保证在 GET 获得键值后保证该键值不被其他客户端修改，直到函数执行完成后才允许其他客户端修改该键键值，这样也可以防止竞态条件，所以使用watch监听某一个key,一旦其中有一个键被修改（或删除），之后的事务就不会执行。