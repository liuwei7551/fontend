# 基于 Gitlab 交付 Vue 的 Docker 镜像

* https://docs.gitlab.com/ce/ci/docker/using_docker_build.html
参考文档
*https://gitlab.com/liuwei7551/vue-ci-docker.git
项目地址

Gitlab 提供了完整 CI/CD 功能并且集成了 docker 镜像服务, 可以在此基础上快速实现 docker 镜像交付.

## Dockerfile

编写 Dockerfile:

```
FROM nginx
COPY dist /usr/share/nginx/html
COPY nginx/server.conf /etc/nginx/conf.d
```

## Gitlab registry

Gitlab 内集成了 registry 可以为所有项目提供 docker 镜像服务. 

```
npm install
docker login # 输入用户名密码登陆
docker built -t registry.gitlab.com/group/project -f Dockerfile . # 编译 Docker 镜像
docker push registry.gitlab.com/group/project # 把 docker 镜像发布到 gitlab registry
```

本地测试docker环境编译执行:

```
npm install
docker build -t gitlab-docker-demo .
docker run gitlab-docker-demo
```

## Gitlab CI 配置

CI 配置文件 `.gitlab-ci.yml` 内容如下

```
image: node

stages:
  - install
  - lint
  - build
  - docker
  - deploy
  
cache:
  paths:
    - node_modules/

install:
  stage: install
  script: 
    - npm install
    
lint:
  stage: lint
  script:
    - npm run lint
    
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
      - nginx/server.conf
  only:
    - docker-ci
    
docker:
  stage: docker
  image: docker:git
  services:
  - name: docker:dind
    alias: docker
  variables:
    DOCKER_DRIVER: overlay2
  before_script:
  - docker info
  script:
  - docker login -u ${USER_NAME} -p ${PASSWORD} ${REGISTRY_ADDRESS}
  - docker build -t ${REGISTRY_ADDRESS}/${USER_NAME}/${PROJECT_NAME} .
  - docker push ${REGISTRY_ADDRESS}/${USER_NAME}/${PROJECT_NAME}

deploy:
  stage: deploy
  script:
  - echo "deploy"
```

## 部署

在服务器上登陆 docker registry:

```
docker login ${REGISTRY_ADDRESS}
```

登陆成功后拉取 docker 镜像:

```
# 以测试服务器为例, 正式服务器需指定相应版本
docker pull ${REGISTRY_ADDRESS}/${USER_NAME}/${PROJECT_NAME}
```

运行:

```
docker run -d -v /var/log/nginx/:/var/log/nginx/ -p 80:80 --restart unless-stopped ${REGISTRY_ADDRESS}/${USER_NAME}/${PROJECT_NAME}
```
