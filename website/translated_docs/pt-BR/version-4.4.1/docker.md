---
id: version-4.4.1-docker
title: Docker
original_id: docker
---

![alt Docker Pulls Count](https://dockeri.co/image/verdaccio/verdaccio "Docker Pulls Count")


Para fazer o pull do último pré-build[docker image](https://hub.docker.com/r/verdaccio/verdaccio/):

```bash
docker pull verdaccio/verdaccio
```

![Docker pull](assets/docker_verdaccio.gif)

<div id="codefund">''</div>

## Versões com Tags

Desde a versão `v2.x`, você pode fazer o pull das imagens pela [tag](https://hub.docker.com/r/verdaccio/verdaccio/tags/), por exemplo:

Para uma versão principal:

```bash
docker pull verdaccio/verdaccio:4
```
Para uma versão secundária:

```bash
docker pull verdaccio/verdaccio:4.0
```

Para uma versão específica (patch):

```bash
docker pull verdaccio/verdaccio:4.0.0
```

> Se você está interessado em uma lista de tags, por favor [acesse o Docker Hub](https://hub.docker.com/r/verdaccio/verdaccio/tags/).

## Executando o Verdaccio usando Docker

Para executar o container do docker:

```bash
docker run -it --rm --name verdaccio -p 4873:4873 verdaccio/verdaccio
```

The last argument defines which image to use. The above line will pull the latest prebuilt image from dockerhub, if you haven't done that already.

Se você tiver [uma imagem construída localmente](#build-your-own-docker-image) use `verdaccio` como o último argumento.


You can use `-v` to bind mount `conf`, `storage` and `plugins` to the hosts filesystem:

```bash
V_PATH=/path/for/verdaccio; docker run -it --rm --name verdaccio \
  -p 4873:4873 \
  -v $V_PATH/conf:/verdaccio/conf \
  -v $V_PATH/storage:/verdaccio/storage \
  -v $V_PATH/plugins:/verdaccio/plugins \
  verdaccio/verdaccio
```
> Note: Verdaccio runs as a non-root user (uid=10001) inside the container, if you use bind mount to override default, you need to make sure the mount directory is assigned to the right user. In above example, you need to run `sudo chown -R 10001:65533 /opt/verdaccio` otherwise you will get permission errors at runtime. [Use docker volume](https://docs.docker.com/storage/volumes/) is recommended over using bind mount.

Verdaccio 4 provides a new set of environment variables to modify either permissions, port or http protocol. Here the complete list:

| Propriedade           | padrão           | Descrição                                                    |
| --------------------- | ---------------- | ------------------------------------------------------------ |
| VERDACCIO_APPDIR      | `/opt/verdaccio` | o diretório de trabalho do docker                            |
| VERDACCIO_USER_NAME | `verdaccio`      | o usuário do sistema                                         |
| VERDACCIO_USER_UID  | `10001`          | o Id do usuário sendo usado para aplicar permissões de pasta |
| VERDACCIO_PORT        | `4873`           | a porta do verdaccio                                         |
| VERDACCIO_PROTOCOL    | `http`           | o protocolo http padrão                                      |



### Plugins
Plugins can be installed in a separate directory and mounted using Docker or Kubernetes, however make sure you build plugins with native dependencies using the same base image as the Verdaccio Dockerfile.

```docker
FROM verdaccio/verdaccio

USER root

ENV NODE_ENV=production

RUN npm i && npm install verdaccio-s3-storage

USER verdaccio
```

### Configuração de Docker e porta customizada
Any `host:port` configured in `conf/config.yaml` under `listen` **is currently ignored when using docker**.

If you want to reach Verdaccio docker instance under different port, lets say `5000` in your `docker run` command add the environment variable `VERDACCIO_PORT=5000` and then expose the port `-p 5000:5000`.

```bash
V_PATH=/path/for/verdaccio; docker run -it --rm --name verdaccio \
  -e "VERDACCIO_PORT=8080" -p 8080:8080 \  
  verdaccio/verdaccio
```

Of course the numbers you give to `-p` paremeter need to match.

### Usando HTTPS com Docker
You can configure the protocol verdaccio is going to listen on, similarly to the port configuration. You have to overwrite the default value("http") of the `PROTOCOL` environment variable to "https", after you specified the certificates in the config.yaml.

```bash
docker run -it --rm --name verdaccio \
  --env "VERDACCIO_PROTOCOL=https" -p 4873:4873
  verdaccio/verdaccio
```

### Usando docker-compose

1. Obtenha a última versão do [docker-compose](https://github.com/docker/compose).
2. Construa e execute o contêiner:

```bash
$ docker-compose up --build
```

You can set the port to use (for both container and host) by prefixing the above command with `VERDACCIO_PORT=5000`.


```yaml
version: '3.1'

services:
  verdaccio:
    image: verdaccio/verdaccio
    container_name: "verdaccio"
    networks:
      - node-network
    environment:
      - VERDACCIO_PORT=4873
    ports:
      - "4873:4873"
    volumes:
      - "./storage:/verdaccio/storage"
      - "./config:/verdaccio/conf"
      - "./plugins:/verdaccio/plugins"  
networks:
  node-network:
    driver: bridge
```

Docker will generate a named volume in which to store persistent application data. You can use `docker inspect` or `docker volume inspect` to reveal the physical location of the volume and edit the configuration, such as:

```bash
$ docker volume inspect verdaccio_verdaccio
[
    {
        "Name": "verdaccio_verdaccio",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/verdaccio_verdaccio/_data",
        "Labels": null,
        "Scope": "local"
    }
]

```

## Construa a sua própria imagem Docker

```bash
docker build -t verdaccio .
```

There is also an npm script for building the docker image, so you can also do:

```bash
yarn run build:docker
```

Note: The first build takes some minutes to build because it needs to run `npm install`, and it will take that long again whenever you change any file that is not listed in `.dockerignore`.

Please note that for any of the above docker commands you need to have docker installed on your machine and the docker executable should be available on your `$PATH`.

## Exemplos de Docker

There is a separate repository that hosts multiple configurations to compose Docker images with `verdaccio`, for instance, as reverse proxy:

[https://github.com/verdaccio/docker-examples](https://github.com/verdaccio/docker-examples)

## Construções de Docker Personalizadas

> If you have made an image based on Verdaccio, feel free to add it to this list.

* [docker-verdaccio-gitlab](https://github.com/snics/docker-verdaccio-gitlab)
* [docker-verdaccio](https://github.com/deployable/docker-verdaccio)
* [docker-verdaccio-s3](https://github.com/asynchrony/docker-verdaccio-s3) Contêiner NPM privado que pode fazer backup para s3
* [docker-verdaccio-ldap](https://github.com/snadn/docker-verdaccio-ldap)
* [verdaccio-ldap](https://github.com/nathantreid/verdaccio-ldap)
* [verdaccio-compose-local-bridge](https://github.com/shingtoli/verdaccio-compose-local-bridge)
* [docker-verdaccio](https://github.com/Global-Solutions/docker-verdaccio)
* [verdaccio-docker](https://github.com/idahobean/verdaccio-docker)
* [verdaccio-server](https://github.com/andru255/verdaccio-server)
* [coldrye-debian-verdaccio](https://github.com/coldrye-docker/coldrye-debian-verdaccio) imagem docker que executa o verdaccio a partir do coldrye-debian-nodejs.