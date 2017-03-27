# Spring Integration Redis lock - lock expiry issue

This is a minimal project to demonstrate lock expiry issue using `RedisLockRegistry` based `LockRegistryLeaderInitiator`.
The registry which obtains the lock does not release it after the expiry in the underlying store which prevents leadership from being revoked and subsequently allows multiple instances becoming leaders. 

The project assumes there is a Redis instance running on standard `localhost:6379`.

To demonstrate the issue, example flow from [Spring Integration Java DSL reference](https://github.com/spring-projects/spring-integration-java-dsl/wiki/spring-integration-java-dsl-reference#example-configurations) is used.
`RedisLockRegistry` is configured with lock TTL of 10 seconds. Start two instances of the application and observe the first instance obtaining the lock and electing itself as leader:

```
2017-03-27 21:47:19.311  INFO 25630 --- [ck-leadership-0] o.s.integration.leader.DefaultCandidate  : DefaultCandidate{role=leader, id=b3ef1b70-1e0a-4264-9346-9aa3efa94105} has been granted leadership; context: LockContext{role=leader, id=b3ef1b70-1e0a-4264-9346-9aa3efa94105, isLeader=true}
```

After 10 seconds the lock in Redis expires but the registry does not release it, while the other instance is able to write the lock and elect itself as leader:

```
2017-03-27 21:47:29.378  INFO 25682 --- [ck-leadership-0] o.s.integration.leader.DefaultCandidate  : DefaultCandidate{role=leader, id=2c3eca9f-4f3f-4588-809c-6aca71ee922e} has been granted leadership; context: LockContext{role=leader, id=2c3eca9f-4f3f-4588-809c-6aca71ee922e, isLeader=true}
```

After 10 seconds the lock from second instance expires leaving both instances as leaders but with no lock persisted in Redis.
