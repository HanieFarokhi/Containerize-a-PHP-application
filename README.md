# PHP Application Deployment with GitLab CI/CD, Docker, and Docker Compose

This repository contains the setup for building, tagging, and deploying a PHP 8.2 application with Apache using GitLab CI/CD, Docker, and Docker Compose.

---

## Overview

- **CI/CD pipeline** to automate build and deployment
- **Dockerfile** for PHP 8.2 Apache container
- **Docker Compose** configuration for container orchestration
- Deployment to a **remote Docker host**

---

## 1. GitLab CI/CD Pipeline

Create a `.gitlab-ci.yml` file in the root of your repo:

```yaml
stages:
  - build
  - deploy

variables:
  IMAGE_NAME: "sw1-app"
  DOCKER_TAG: "latest"
  CONTAINER_NAME: "sw1-app"
  DATE: "$(date +%Y%m%d)"
  DOCKER_HOST: "tcp://docker:2375"
  DOCKER_TLS_CERTDIR: ""
  DOCKER_DRIVER: overlay2

build_sanbox_sw1:
  stage: build
  image: $REPOSITORY:8082/docker:latest
  services:
    - name: $REPOSITORY:8082/docker:dind
      alias: docker
      command: 
        - "--tls=false"
        - "--insecure-registry=$REGISTRY:5000"

  before_script:
    - docker login http://$REGISTRY:5000 -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD"
  script:
    - DATE=$(date +%Y%m%d)
    - FULL_TAG="${IMAGE_NAME}-${DATE}"
    - echo "Generated Docker Tag:$FULL_TAG"
    - docker build -t $REGISTRY/$IMAGE_NAME:$FULL_TAG .
    - docker push $REGISTRY/$IMAGE_NAME:$FULL_TAG
    - echo "$FULL_TAG" > image_tag.txt
  artifacts:
    paths:
      - image_tag.txt
    expire_in: 1 hour
  only:
    - main
  tags:
    - build-runner

Deploy_sanbox_sw1:
  stage: deploy
  image: $REPOSITORY:8082/docker:latest   
  variables:
    DOCKER_HOST: "tcp://$APP-IP:2375"  
    DOCKER_TLS_CERTDIR: ""
  script:
    - APP_TAG=$(cat image_tag.txt)
    - sed "s|\${APP_TAG}|$APP_TAG|g" docker-compose.sanbox.sw1.yml > docker-compose.tmp.yml
    - echo "Deploying image:$REGISTRY/$IMAGE_NAME:$APP_TAG to Docker host $DOCKER_HOST"
    - docker compose -f docker-compose.tmp.yml pull
    - docker compose -f docker-compose.tmp.yml down
    - docker compose -f docker-compose.tmp.yml up -d --remove-orphans
  dependencies:
    - build_sanbox_sw1
  only:
    - main
  when: manual
