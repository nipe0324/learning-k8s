# Fargate Hello

Flask on AWS Fargate by the post: https://qiita.com/oyngtmhr/items/45a0d3158e6dccb0882d


## Run docker conainer on localhost

```
$ cd fargate_hello/flask
$ docker build -t flask-fargate-hello:latest .
$ docker run -p 80:5000 flask-fargate-hello
$ curl localhost
Flask on AWS Fargate
```

## Run docker container on AWS Fargate

https://qiita.com/oyngtmhr/items/45a0d3158e6dccb0882d
