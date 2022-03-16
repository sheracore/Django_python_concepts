# Django_python_concepts

# Celery
## redis
```
redis is nosql database that store data in key value format also you can use it subscribe/publish (stack) tool by running subscribe(consumer) on a shell and publish(prducer) data in another shell and see result in subscribe shell

---> Shell number 1
redis-cli                                         [8/18]
127.0.0.1:6379> SUBSCRIBE first channel_name
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "first"
3) (integer) 1
1) "message"
2) "channel_name"
3) "hello"

---> shell number 2
redis-cli
127.0.0.1:6379> PUBLISH channel_name hello
(integer) 1



```
