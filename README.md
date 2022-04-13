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
### Each cpu core has two thread so each worker on any cpu can (subscribe)consume two concurency task
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
## If you want to bind a task after running another task in other word should run first task so second task can run, use chain or pip
```
from celery import chain
chain(debug_task.si(1), debug_task.si(2)).apply_async()
chain(debug_task.si(1), debug_task.si(2)).delay()

(debug_task.si(1, 2) | debug_task.si(3)).apply_async()
(debug_task.si(1, 2) | debug_task.si(3)).delay()
```
### And if you want to use result of first task to second task use "s" 
```
@task()
def add(a, b):
    time.sleep(5) # simulate long time processing
    return a + b
    
# import chain from celery import chain
# the result of the first add job will be 
# the first argument of the second add job
ret = chain(add.s(1, 2), add.s(3)).apply_async()

# another way to express a chain using pipes
ret2 = (add.s(1, 2) | add.s(3)).apply_async()
```

## In our app samples
```
chain(debug_task.si(1),debug_task.si(2)).delay()
(debug_task.si(2,3) | debug_task.si(3,4)).delay()
a = (debug_task.s(2,3) | debug_task.s(4)).delay()
debug_task.s(2,3).delay()
a = (debug_task.s(2,3) | debug_task.s(4) | debug_task.s(5)).delay()
a.parent.parent.id
group(debug_task.si(1), debug_task.si(2)).delay()
a = group(debug_task.si(1,2), debug_task.si(2,4)).delay()
a = (group(debug_task.si(1,2), debug_task.si(2,4))|group(debug_task.si(1,2), debug_task.si(2,4))).delay()
a.parent.children[0].state
```

## Work with redis in python celery
```
app = Celery('viruspod')
""" 'viruspod' is name of queue """
i = app.control.inspect()
i.registerd() # All methos that are defined as task
i.active() # active tasks mention tasks that have cpu core and in progress
i.reserved() # that tasks os ready to run
```

## There is a problem, we cant see all tasks in redis queue by inspect so we should use python redis
```
import redis
r = redis.Redis()
r.lrange(queue, min, max) # for lists
r.llen queue # length of list
r.lrange(quere, 0, llen(queue))
```

## In worker command line and delay a task we can define queue instead of default queue
## Also if we don't want having reserved tasks we can use prefetch-multiplier switch in worker command line
```
celery -A proj worker --prefetch-multiplier=1 -l info -Q scan-sessions
tasks.append(debug_task.si(i).set(queue='scan-sessions'))
```
## Some golden points.
#### 1) If you run some group continuous that are run continuous and that are arranged but in group that tasks run together(parallel) based on the number of cup cores.

#### 2) To disable reserved tasks in queue you should first add --prefetch-multiplier=1 to worker command line and add following envs in settings:
```
celery -A proj worker --prefetch-multiplier=1 -l info -Q scan-sessions --concurrency=1
CELERYD_PREFETCH_MULTIPLIER = 1
CELERY_TASK_TRACK_STARTED = True
```

### To get task objects inside of outer object shoudl using children method
```
@shared_task
def task1():
    task2.delay()

a task1.delay()
a.children

or

sig = task1.si | task2.si
sig.apply_aysn()

sig.id is for task2 and sig.parent.id is for task1
```



