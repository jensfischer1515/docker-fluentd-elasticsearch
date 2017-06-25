# docker-fluentd-elasticsearch
Docker image for fluentd with Elasticsearch plugin

## Build image

Use `docker build` command to build the image.

```
$ docker build -t jensfischerhh/docker-fluentd-elasticsearch:latest ./
```

## Test

Once the image is built, it's ready to run.
Following commands run fluentd sharing `./log` directory with the host machine:

```
$ mkdir -p log
$ docker run -it --rm --name fluentd -v $(pwd)/log:/fluentd/log jensfischerhh/docker-fluentd-elasticsearch:latest

2017-06-25 18:35:35 +0000 [info]: reading config file path="/fluentd/etc/fluent.conf"
2017-06-25 18:35:35 +0000 [info]: using configuration file: <ROOT>
  <source>
    @type forward
  </source>
  <match **>
    @type stdout
  </match>
</ROOT>
2017-06-25 18:35:35 +0000 [info]: starting fluentd-0.14.18 pid=22
2017-06-25 18:35:35 +0000 [info]: spawn command to main:  cmdline=["/usr/bin/ruby2.3", "-Eascii-8bit:ascii-8bit", "/usr/local/bin/fluentd", "-c", "/fluentd/etc/fluent.conf", "-p", "/fluentd/plugins", "--under-supervisor"]
2017-06-25 18:35:35 +0000 [info]: gem 'fluent-plugin-elasticsearch' version '1.9.5'
2017-06-25 18:35:35 +0000 [info]: gem 'fluentd' version '0.14.18'
2017-06-25 18:35:35 +0000 [info]: adding match pattern="**" type="stdout"
2017-06-25 18:35:35 +0000 [info]: adding source type="forward"
2017-06-25 18:35:35 +0000 [info]: #0 starting fluentd worker pid=26 ppid=22 worker=0
2017-06-25 18:35:35 +0000 [info]: #0 listening port port=24224 bind="0.0.0.0"
2017-06-25 18:35:35 +0000 [info]: #0 fluentd worker is now running worker=0
2017-06-25 18:35:35.503925847 +0000 fluent.info: {"worker":0,"message":"fluentd worker is now running worker=0"}
```

Open another terminal and type following command to inspect IP address.
Fluentd is running on this IP address:

```
$ docker inspect -f '{{.NetworkSettings.IPAddress}}' fluentd

172.17.0.2
```

Let's try to use another docker container to send its logs to Fluentd.

```
$ docker run --log-driver=fluentd --log-opt tag="docker.{{.ID}}" --log-opt fluentd-address=172.17.0.2:24224 python:alpine echo Hello
# and force flush buffered logs
$ docker kill -s USR1 fluentd

2017-06-25 18:38:03.000000000 +0000 docker.0042198d3675: {"source":"stdout","log":"Hello","container_id":"0042198d3675bf437c5f92d90fc80212b7f677c14d5a22607730d9a08a9baf67","container_name":"/elated_meitner"}
```
