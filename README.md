# Docker Infrastructure

## Commands:
Notes:
before using this commands please change `$prject_root` to real project root,
change `{{servie_name}}` to your sevice name, change `prject_name` to projet name

Build docker image
```sh
docker build -f $prject_root/../docker_infrastructure/dockerfiles/ruby/3.2.4/dev/Dockerfile --platform=linux/amd64 -t $prject_name:dev .
```
Run docker container

```sh
docker exec -u=0 -it $(docker ps | grep {{servie_name}} | awk '{ print $1 }') /bin/bash
```
