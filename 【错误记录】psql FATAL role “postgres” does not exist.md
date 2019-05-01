# 【错误记录】psql: FATAL: role “postgres” does not exist

这是因为 `psql` 默认是连接的当前用户名的数据库，字面意思就是当前用户名的数据库不存在，当然，`PostgreSQL` 默认会创建有三个数据库

```
postgres
template0
template1
```

如果想连接当前用户名数据库，可以通过

```
createdb
```

这条命令是默认创建一个以当前用户名为名称的数据，执行之后，就能通过 `psql`默认连接了

