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

## Celery worker
### You can run a worker by this command
```
celery -A proj worker -l info
```
### Each cpu core has two thread so each worker on any cpu can (subscribe)consumer two concurency
### So if you run a worker on i7 cpu worker can get 16 published message or task concurency
### And if you want worker just consume on one(two, three, ..)thread you can use
```
celery -A proj worker -l info --concurrency=1
celery -A proj worker -l info --concurrency=2
```


## This is a celery task
```
@app.task(bind=True)
def debug_task(self, tid):
    logger.info('testing celery log')
    return tid
```
## Publish a task
```
debug_task.delay(i)
or
debug_task.si(1).apply_async()
```

## If you want to publish two or more tasks together concurncy use group
```
from celery import group
group(debug_task.si(1,2), debug_task.si(2,4)).delay()
```
