# 基于 Gitlab 交付 Vue 的 Docker 镜像

* https://docs.gitlab.com/ce/ci/docker/using_docker_build.html

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
docker built -t registry.gitlab.example.com/group/project -f Dockerfile . # 编译 Docker 镜像
docker push registry.gitlab.example.com/group/project # 把 docker 镜像发布到 gitlab registry
```

本地测试docker环境编译执行:

```
npm install
docker build -t gitlab-go-docker-demo docker/test/Dockerfile .
docker run gitlab-go-docker-demo
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
    DOCKER_HOST: tcp://localhost:2375
  before_script:
  - docker info
  script:
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  - docker build -t $CI_REGISTRY_IMAGE/$CI_COMMIT_REF_NAME .
  - docker push $CI_REGISTRY_IMAGE/$CI_COMMIT_REF_NAME:latest

deploy:
  stage: deploy
  script:
  - echo "deploy"
```

## 部署

在服务器上登陆 docker registry:

```
docker login
```

登陆成功后拉取 docker 镜像:

```
# 以测试服务器为例, 正式服务器需指定相应版本
docker pull $CI_REGISTRY_IMAGE/$CI_COMMIT_REF_NAME:latest
```

运行:

```
docker run -p 80:80 -v /var/log/:/var/log/ --restart unless-stopped registry.gitlab.example.com/group/project/test
```
