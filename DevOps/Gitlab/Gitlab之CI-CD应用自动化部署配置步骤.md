---
title: Gitlab 之 CI/CD应用自动化部署配置步骤
date: 2020-06-16 15:44:34
---

### 配置 Runner

Runner 用来运行 Pipeline，Runner 可以是 ssh、docker 等类型，推荐使用隔离性更好的 docker。默认已经配置好一个所有项目公用的 Shared Runner，如有需要可为 Group 和 Project 创建单独的 Runner。

### 配置 Variables

有些值，比如 Docker Registry 帐号、Kubernetes 集群访问密钥等，不方便直接写死在 Pipeline 定义文件中，可现在 Group 或 Project 上定义好。GitLab 本身已内置许多变量 [Predefined environment variables reference](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)。

### 定义 Pipeline

Pipeline 用来描述持续集成和持续部署的具体过程，它由一个个顺序执行的 Stage 构成，每个 Stage 包含一到多个并行执行的 Job。下面是一个 Java Spring Boot 服务的 Pipeline 示例：

```yaml
stages:
  - build
  - package
  - deploy

maven-build:
  stage: build
  only:
    refs:
      - dev
      - test
      - master
  image: registry.prod.bbdops.com/common/maven:3.6.3-jdk-8
  variables:
    MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository -Dmaven.test.skip=true "
    MAVEN_CLI_OPTS: "-s .m2/settings.xml --batch-mode"
  script:
    - mvn $MAVEN_CLI_OPTS package
  artifacts:
    paths:
      - target/*.jar
  cache:
    paths:
      - .m2/repository/

docker-build:
  stage: package
  image: registry.prod.bbdops.com/common/docker:19.03.8
  services:
    - name: registry.prod.bbdops.com/common/docker:19.03.8-dind
      alias: docker
      command: ["--insecure-registry=registry.prod.bbdops.com", "--registry-mirror=https://nypkinfs.mirror.aliyuncs.com"]
  variables:
    DOCKER_IMAGE_NAME: appone/canghai-user
  script:
    - echo $BBD_DOCKER_REGISTRY_PASSWORD | docker login -u $BBD_DOCKER_REGISTRY_USERNAME --password-stdin $BBD_DOCKER_REGISTRY
    - docker pull $BBD_DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:latest || true
    - docker build --cache-from $BBD_DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:latest -t $BBD_DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:$CI_COMMIT_SHORT_SHA -t $BBD_DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:$CI_COMMIT_BRANCH -t $BBD_DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:latest .
    - docker push $BBD_DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:$CI_COMMIT_SHORT_SHA
    - docker push $BBD_DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:$CI_COMMIT_BRANCH
    - docker push $BBD_DOCKER_REGISTRY/$DOCKER_IMAGE_NAME:latest

.kubernetes-deploy:
  stage: deploy
  image: registry.prod.bbdops.com/common/google/cloud-sdk:289.0.0
  script:
    - sed -i 's/$CI_COMMIT_SHORT_SHA/'"$CI_COMMIT_SHORT_SHA"'/' deployment.yml
    - kubectl apply -f deployment.yml -n canghai

kubernetes-deploy-development:
  extends: .kubernetes-deploy
  only:
    refs:
      - dev
  before_script:
    - cat $KUBERNETES_DEVELOPMENT_CLUSTER_CONFIG >~/.kube/config

kubernetes-deploy-testing:
  extends: .kubernetes-deploy
  only:
    refs:
      - test
  before_script:
    - cat $KUBERNETES_TESTING_CLUSTER_CONFIG >~/.kube/config

kubernetes-deploy-production:
  extends: .kubernetes-deploy
  only:
    refs:
      - master
  when: manual
  before_script:
    - cat $KUBERNETES_PRODUCTION_CLUSTER_CONFIG >~/.kube/config
```

如果是前端项目，可替换其中的 `maven-build` 任务为如下的 `node-build`。

```yaml
node-build:
  stage: build
  only:
    refs:
      - dev
      - test
      - master
  image: registry.prod.bbdops.com/common/node:12.16.3
  variables:
    CACHE_FOLDER: .yarn
  script:
    - yarn config set cache-folder $CACHE_FOLDER
    - yarn install --registry http://verdaccio.bbdops.com/
    - yarn run build
  artifacts:
    paths:
      - build/
  cache:
    paths:
      - $CACHE_FOLDER
```

### 触发 Pipeline

Push 代码到某个分支即可自动触发跟该分支相关的 Job，也可在项目 CI/CD 页手动触发 Pipeline 或者重试某个 Job，还可创建 Schedule 来定时触发。