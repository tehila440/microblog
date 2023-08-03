`docker run --name microb -d -p 8000:5000 --rm tehila440/microblog:latest` <br>
`http://localhost:8000`<br>
start redis service `redis-server`<br>
run worker `rq worker microblog-tasks`<br>
`>>> from redis import Redis`<br>
`>>> import rq`<br>
`>>> queue = rq.Queue('microblog-tasks', connection=Redis.from_url('redis://'))`<br>
`>>> job = queue.enqueue('app.tasks.example', 23)`<br>
`>>> job.get_id()`<br>
'c651de7f-21a8-4068-afd5-8b982a6f6d32'`<br>
