# gitlab

# Install on a VM
```
# 1. Install and configure the necessary dependencies
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl

sudo apt-get install -y postfix

# 2. Add the GitLab package repository and install the package 
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash

# Get intial credential  
cat /etc/gitlab/initial_root_password

# Connect as root
root / $(cat /etc/gitlab/initial_root_password)

# set demo dns
echo "192.168.1.200 gitlab.example.com" >> /etc/hosts

# Install gitlab
apt-get install gitlab-ee

# Configure
gitlab-ctl reconfigure
```

## Register a runner
```
# Generate a registration runner token here: 
* http://gitlab.example.com/admin/runners
```
<img width="1604" alt="image" src="https://user-images.githubusercontent.com/32338685/228361286-4d82d080-ef13-4179-9144-23827b174851.png">

```
Download the package here: https://docs.gitlab.com/runner/install/linux-manually.html
dpkg -i 


gitlab-runner register --locked=false

Runtime platform                                    arch=amd64 os=linux pid=5414 revision=456e3482 version=15.10.0
WARNING: Running in user-mode.                     
WARNING: The user-mode requires you to manually start builds processing: 
WARNING: $ gitlab-runner run                       
WARNING: Use sudo for system-mode:                 
WARNING: $ sudo gitlab-runner...                   
                                                   
WARNING: There might be a problem with your config
jsonschema: '/runners' does not validate with https://gitlab.com/gitlab-org/gitlab-runner/common/config#/$ref/properties/runners/type: expected array, but got null 
Enter the GitLab instance URL (for example, https://gitlab.com/):
http://gitlab.example.com/
Enter the registration token:
xxxxxxxx
Enter a description for the runner:
[ubuntu2004]: gitlab.example.com runner1
Enter tags for the runner (comma-separated):
runner1
Enter optional maintenance note for the runner:
in maintenance
WARNING: Support for registration tokens and runner parameters in the 'register' command has been deprecated in GitLab Runner 15.6 and will be replaced with support for authentication tokens. For more information, see https://gitlab.com/gitlab-org/gitlab/-/issues/380872 
Registering runner... succeeded                     runner=kPvs6hax
Enter an executor: ssh, virtualbox, docker+machine, docker-ssh+machine, docker-ssh, parallels, shell, instance, kubernetes, custom, docker:
docker
Enter the default Docker image (for example, ruby:2.7):
ruby:2.7
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
 
Configuration (with the authentication token) was saved in "/home/ubuntu/.gitlab-runner/config.toml" 



gitlab-runner verify
```
## Runner Config File
* /etc/gitlab-runner/config.toml

## CI/CD
* Basic CI/CD workflow with Gitlab CI
* What is CI / CD?

### Continious Integration
* Code -> CI pipleine ( Build -> Code Quality -> Test -> Package)

### Continious Delivery
* Continious Delivery= previous stages you have create a package, auto install this package in an ENV decide if a specific package should go in PRD
* CD pipeline ( Review / Test -> Stagging -> questions? -> Production )

### Manage branch rights
* Settings => Repository => Protected Branches => expand

### Can see the differents environements

<img width="1166" alt="image" src="https://user-images.githubusercontent.com/32338685/189135819-03238ec3-964e-41a5-ad6f-a1bda3b7063d.png">

Deployments => Environments

## Lynter

<img width="1536" alt="image" src="https://user-images.githubusercontent.com/32338685/189140419-8a336589-5afb-422b-9c48-f4fc7fba0fd0.png">

## Yaml syntax help doc
* https://github.com/davidboukari/yamljson/blob/master/README.md
* Yaml converter: https://codebeautify.org/yaml-to-json-xml-csv

## Disabling a job
* Just add . before the name ex .Deploy Production:

## Anchor in yaml file ex:
```
use anchor &name value
self: *name
```

## Sample
* https://gitlab.com/zzdanilou/car-asembly-line
* Create file .gitlab-ci.yml
* There is a sample with anchor &deploy for a template
```
stages:
  - build
  - test
  - deploy review
  - deploy staging
  - staging test
  - deploy production
  - production test

image: node
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

variables:
  STAGING_DOMAIN: instazone-staging.surge.sh
  PRODUCTION_DOMAIN: instazone.surge.sh

Build Web Site:
  stage: build
  before_script:
    - echo "Build Web Site"
  script:
    - echo "CI_COMMIT_SHORT_SHA=$CI_COMMIT_SHORT_SHA"
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
    - sed -i "s/%%VERSION%%/$CI_COMMIT_SHORT_SHA/" ./public/index.html
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public

Test the artifact:
  stage: test
  image: alpine
  before_script:
    - echo "Test the artifact"
  script:
  - grep -q "Gatsby" ./public/index.html

Test Web Site:
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve &
    - sleep 10
    - echo "curl http://localhost:9000 | grep -q "Gatsby" ./public/index.html"

.deploy_template: &deploy
  only:
    - master
  environment:
    url: http://$DOMAIN
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${DOMAIN}"

Deploy review:
  <<: *deploy
  stage: deploy review
  only:
    - merge_requests
  variables:
    DOMAIN: instazone-${CI_ENVIRONMENT_SLUG}.surge.sh  
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: http://${DOMAIN}
    on_stop: Stop review
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${DOMAIN}"

Stop review:
  stage: deploy review
  variables:
   GIT_STRATEGY: none
  when: manual
  only:
    - merge requests
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop 
  script:
     - npm install --global surge
     - echo "surge teardown instazone-${CI_ENVIRONMENT_SLUG}.surge.sh"  

Deploy staging: 
  stage: deploy staging
  environment:
    name: staging
    url: $STAGING_DOMAIN
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${url}"

Staging test:
  stage: staging test
  image: alpine
  environment:
    name: staging
    url: $STAGING_DOMAIN   
  script: 
    - apk add --no-cache curl
    - echo "curl http://${url}| grep 'Hi people'"
    - echo "curl http://${url}| grep $CI_COMMIT_SHORT_SHA" 


Deploy production: 
  stage: deploy production
  when: manual
  allow_failure: false
  only: 
    - master
  environment:
    name: production
    url: $PRODUCTION_DOMAIN
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${url}"


Production test:
  stage: production test
  image: alpine
  only: 
    - master
  environment:
    name: production
    url: $PRODUCTION_DOMAIN  
  script: 
    - apk add --no-cache curl
    - echo "curl http://${PRODUCTION_DOMAIN}| grep 'Hi people'"
    - echo "curl http://${PRODUCTION_DOMAIN}| grep $CI_COMMIT_SHORT_SHA"     

```



```
stages:
  - build
  - test
  - deploy review
  - deploy staging
  - staging test
  - deploy production
  - production test

image: node
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

variables:
  STAGING_DOMAIN: instazone-staging.surge.sh
  PRODUCTION_DOMAIN: instazone.surge.sh

Build Web Site:
  stage: build
  before_script:
    - echo "Build Web Site"
  script:
    - echo "CI_COMMIT_SHORT_SHA=$CI_COMMIT_SHORT_SHA"
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
    - sed -i "s/%%VERSION%%/$CI_COMMIT_SHORT_SHA/" ./public/index.html
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public

Test the artifact:
  stage: test
  image: alpine
  before_script:
    - echo "Test the artifact"
  script:
  - grep -q "Gatsby" ./public/index.html

Test Web Site:
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve &
    - sleep 10
    - echo "curl http://localhost:9000 | grep -q "Gatsby" ./public/index.html"

Deploy review:
  stage: deploy review
  only:
    - merge_requests
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: http://instazone-${CI_ENVIRONMENT_SLUG}.surge.sh
    on_stop: Stop review
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${url}"

Stop review:
  stage: deploy review
  variables:
   GIT_STRATEGY: none
  when: manual
  only:
    - merge requests
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop 
  script:
     - npm install --global surge
     - echo "surge teardown instazone-${CI_ENVIRONMENT_SLUG}.surge.sh"  

Deploy staging: 
  stage: deploy staging
  environment:
    name: staging
    url: $STAGING_DOMAIN
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${url}"

Staging test:
  stage: staging test
  image: alpine
  environment:
    name: staging
    url: $STAGING_DOMAIN   
  script: 
    - apk add --no-cache curl
    - echo "curl http://${url}| grep 'Hi people'"
    - echo "curl http://${url}| grep $CI_COMMIT_SHORT_SHA" 


Deploy production: 
  stage: deploy production
  when: manual
  allow_failure: false
  only: 
    - master
  environment:
    name: production
    url: $PRODUCTION_DOMAIN
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${url}"


Production test:
  stage: production test
  image: alpine
  only: 
    - master
  environment:
    name: production
    url: $PRODUCTION_DOMAIN  
  script: 
    - apk add --no-cache curl
    - echo "curl http://${PRODUCTION_DOMAIN}| grep 'Hi people'"
    - echo "curl http://${PRODUCTION_DOMAIN}| grep $CI_COMMIT_SHORT_SHA"     

```


```
stages:
- build
- test

build the car:
  stage: build
  script:
  - mkdir build
  - cd build
  - touch cat.txt
  - echo "chassis" >> car.txt
  - echo "engine"  >> car.txt
  - echo "wheels"  >> car.txt
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
    - build/

test the car:
  stage: test
  script:
  - ls
  - test -f build/car.txt
  - cd build
  - cat car.txt
  - grep "chassis" car.txt
  - grep "engine"  car.txt
  - grep "wheels"  car.txt
```

## Explications
```
# Set the order of the stages
stages:
- build
- test

# Stage Display name
build the car:
  # Stage step name
  stage: build
  # Script to execute it is a job
  script:
  - mkdir build
  - cd build
  - touch cat.txt
  - echo "chassis" >> car.txt
  - echo "engine"  >> car.txt
  - echo "wheels"  >> car.txt
  # Keep some datas as artefacts can be use in the other stages or downloaded after
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
    - build/

test the car:
  stage: test
  script:
  - ls
  - test -f build/car.txt
  - cd build
  - cat car.txt
  - grep "chassis" car.txt
  - grep "engine"  car.txt
  - grep "wheels"  car.txt



```


## Your first pipeline
```
1st project: Car assembly line
https://gitlab.com/zzdanilou/car-asembly-line

* Create .gitlab-ci.yaml
in script: whith differents actions

build the car:
  script:
   - mkdir build
  - touch cat.txt
  - echo "chassis" > car.txt
  - echo "engine" > car.txt
  - echo "wheels" > car.txt

commit changes

* Add new step  "test the car:"

build the car:
  script:
  - mkdir build
  - touch cat.txt
  - echo "chassis" > car.txt
  - echo "engine" > car.txt
  - echo "wheels" > car.txt
 
test the car:
  script:
  - test -f build/car.txt
  

* Give the order of the steps by using stages:

stages:
- build
- test

stages:
- build
- test
 
build the car:
  stage: build
  script:
  - mkdir build
  - touch cat.txt
  - echo "chassis" > car.txt
  - echo "engine" > car.txt
  - echo "wheels" > car.txt
 
test the car:
  stage: test
  script:
  - test -f build/car.txt
  - cd build
  - grep "chassis" car.txt
  - grep "engine" car.txt
  - grep "wheels" cat.txt

* This gitlab conf is valid !!!

* test stage has failed look at why?

error => exit 1
The file does not exist because it has created on a previous stage, but at the end of the stage it has been removed

artifacts is what we upload at the end of the job stage

15:43
1. Introduction
2. Your first pipeline
Keep directory build 

stages:
- build
- test
 
build the car:
  stage: build
  script:
  - mkdir build
  - touch cat.txt
  - echo "chassis" > car.txt
  - echo "engine" > car.txt
  - echo "wheels" > car.txt
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - build/
 
test the car:
  stage: test
  script:
  - test -f build/car.txt
  - cd build
  - grep "chassis" car.txt
  - grep "engine" car.txt
  - grep "wheels" cat.txt

* build directory exist but grep error in the file car.txt

stages:
- build
- test
 
build the car:
  stage: build
  script:
  - mkdir build
  - cd build
  - touch cat.txt
  - echo "chassis" >> car.txt
  - echo "engine"  >> car.txt
  - echo "wheels"  >> car.txt
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - build/
 
test the car:
  stage: test
  script:
  - ls
  - test -f build/car.txt
  - cd build
  - cat car.txt
  - grep "chassis" car.txt
  - grep "engine"  car.txt
  - grep "wheels"  cat.txt

Menu CI/CD pipeline
It is possible to download the artefact

Click on the job ccan download the artifacts

Can browse and download the artefact:
https://gitlab.com/zzdanilou/car-asembly-line/-/jobs/2989986300
https://gitlab.com/zzdanilou/car-asembly-line/-/jobs/2989986300/artifacts/browse/build/


The pass stages
The arteface is undependant of the git project

https://gitlab.com/zzdanilou/car-asembly-line/-/jobs/2989986300/artifacts/browse/build/

## Create a project
```
https://www.gatsbyjs.com/docs/quick-start/#install-the-gatsby-cli

* Install brew: https://dyclassroom.com/howto-mac/how-to-install-nodejs-and-npm-on-mac-using-homebrew
If errors:  brew update-reset

* Install nodejs : https://nodejs.org/en/
node --version
v16.17.0

npm --version
8.15.0

npm install -g gatsby-cli
gatsby new static-website
cd  static-website
gatsby develop

the web server is started http://localhost:8000


* Create project on Gitlab
Do not initilialize readme.md
git remote add origin git@gitlab.com:zzdanilou/my-static-website.git
git push -u origin master
```

## Create a project with build
```
* Install brew: https://dyclassroom.com/howto-mac/how-to-install-nodejs-and-npm-on-mac-using-homebrew
If errors:  brew update-reset

* Install nodejs : https://nodejs.org/en/
node --versio
v16.17.0

npm --version
8.15.0

npm install -g gatsby-cli
gatsby new static-website
cd  static-website
gatsby develop

the web server is started http://localhost:8000

Create project on Gitlab
Do not initilialize readme.md

git remote add origin git@gitlab.com:zzdanilou/my-static-website.git
git push -u origin master

# Build
gatsby build

cd publi
ls public/
217-97cbec2dfef1c16ad273.js
217-97cbec2dfef1c16ad273.js.LICENSE.txt
217-97cbec2dfef1c16ad273.js.ma
231-35d180c52a3a65770f82.js
231-35d180c52a3a65770f82.js.map
404
404.html
app-aca952eae2657b3a4228.js
app-aca952eae2657b3a4228.js.map
chunk-map.json
component---src-pages-404-js-809cc5ac4c6326673cb1.j
component---src-pages-404-js-809cc5ac4c6326673cb1.js.map
component---src-pages-index-js-82f5f0c7d7f62054c694.j
component---src-pages-index-js-82f5f0c7d7f62054c694.js.map
component---src-pages-page-2-js-a16e4bc91

want to save directory public as artifacts

Docker images
sudo apt-get install -y nodejs
npm install -g gatsby

can easy use diff version of nodejs in the containers

Build Web Site:
  script:
  - npm install
  - npm install -g gatsby-cli
  - gatsby build

hub.docker.io nodejs official
image: nod
Use nodejs docker image

Build Web Site:
  image: node
  script:
  - npm install
  - npm install -g gatsby-cli
  - gatsby build
 
* troubleshooting

Job success
artifacts:
browse artifacts

Build Web Site:
  image: node
  script:
  - npm install
  - npm install -g gatsby-cli
  - gatsby build
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public

* Adding a test stage
Jobs return status:
0 -> Succes
1-255 -> Fail

cd public
# grep quiet mode -q
grep -q "Gatsby" index.html
echo $?

Add grep Error
look at the pipeline

---
stages:
  - build
  - test
 
Build Web Site:
  stage: build
  image: node
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public
 
Test the artifact:
  stage: test
  script:
  - grep "Gatsby" ./public/index.html
  - grep "XXXX" ./public/index.html
 
# --
---
stages:
  - build
  - test
 
Build Web Site:
  stage: build
  image: node
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public
 
Test the artifact:
  stage: test
  script:
  - grep -q "Gatsby" ./public/index.html

# Job in parallal 2 jobs in // in step test 
stages:
  - build
  - test
 
Build Web Site:
  stage: build
  image: node
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public
 
Test the artifact:
  stage: test
  script:
  - grep -q "Gatsby" ./public/index.html
 
Test Web Site:
  image: alpine
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve
    - curl http://localhost:9000 | grep -q "Gatsby" ./public/index.html

* Deployment using surge.sh
# Use surge.sh for the deployment
npm install --global surge

Need to create an account in surge
surge token

# Create Variables
Settings => CI/CD => Variables => Add Variable
=> Key / Value

Protect Variable disable or the var is only available for a branch
Create
SURGE_LOGIN  Disable protect var, Disable Mask var
SURGE_TOKEN Disable Protect var, Enable Mask var (secret min >= 8 car)

```

# Sample
```
stages:
  - build
  - test
  - deploy

image: node

Build Web Site:
  stage: build
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public

Test the artifact:
  stage: test
  image: alpine
  script:
  - grep -q "Gatsby" ./public/index.html

Test Web Site:
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve &
    - sleep 10
    - curl http://localhost:9000 | grep -q "Gatsby" ./public/index.html

deploy to surge: 
  stage: deploy
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain instazone.surge.sh"

```

## Sample
```
stages:
  - build
  - test
  - deploy
  - deployment test

image: node

Build Web Site:
  stage: build
  script:
    - echo "CI_COMMIT_SHORT_SHA=$CI_COMMIT_SHORT_SHA"
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
    - sed -i "s/%%VERSION%%/$CI_COMMIT_SHORT_SHA/" ./public/index.html
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public

Test the artifact:
  stage: test
  image: alpine
  script:
  - grep -q "Gatsby" ./public/index.html

Test Web Site:
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve &
    - sleep 10
    - echo "curl http://localhost:9000 | grep -q "Gatsby" ./public/index.html"

Deploy to surge: 
  stage: deploy
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain instazone.surge.sh"

Test deploy:
  stage: deployment test
  script: 
    - apk add --no-cache curl
    - echo "curl http://instazone.surge.sh| grep 'Hi people'"
    - echo "curl http://instazone.surge.sh| grep $CI_COMMIT_SHORT_SHA" 

```


## Manual for production env
```
stages:
  - build
  - test
  - deploy staging
  - staging test
  - deploy production
  - production test

image: node
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

variables:
  STAGING_DOMAIN: instazone-staging.surge.sh
  PRODUCTION_DOMAIN: instazone.surge.sh

Build Web Site:
  stage: build

  script:
    - echo "CI_COMMIT_SHORT_SHA=$CI_COMMIT_SHORT_SHA"
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
    - sed -i "s/%%VERSION%%/$CI_COMMIT_SHORT_SHA/" ./public/index.html
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public

Test the artifact:
  stage: test
  image: alpine
  script:
  - grep -q "Gatsby" ./public/index.html

Test Web Site:
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve &
    - sleep 10
    - echo "curl http://localhost:9000 | grep -q "Gatsby" ./public/index.html"

Deploy staging: 
  stage: deploy staging
  environment:
    name: staging
    url: $STAGING_DOMAIN
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${url}"

Staging test:
  stage: staging test
  image: alpine
  environment:
    name: staging
    url: $STAGING_DOMAIN    
  script: 
    - apk add --no-cache curl
    - echo "curl http://${url}| grep 'Hi people'"
    - echo "curl http://${url}| grep $CI_COMMIT_SHORT_SHA" 


Deploy production: 
  stage: deploy production
  when: manual
  allow_failure: false
  environment:
    name: production
    url: $PRODUCTION_DOMAIN
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${url}"


Production test:
  stage: production test
  image: alpine
  environment:
    name: production
    url: $PRODUCTION_DOMAIN  
  script: 
    - apk add --no-cache curl
    - echo "curl http://${PRODUCTION_DOMAIN}| grep 'Hi people'"
    - echo "curl http://${PRODUCTION_DOMAIN}| grep $CI_COMMIT_SHORT_SHA"     

```

## Only for some specifiques branches
```
stages:
  - build
  - test
  - deploy staging
  - staging test
  - deploy production
  - production test

image: node
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

variables:
  STAGING_DOMAIN: instazone-staging.surge.sh
  PRODUCTION_DOMAIN: instazone.surge.sh

Build Web Site:
  stage: build

  script:
    - echo "CI_COMMIT_SHORT_SHA=$CI_COMMIT_SHORT_SHA"
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
    - sed -i "s/%%VERSION%%/$CI_COMMIT_SHORT_SHA/" ./public/index.html
  artifacts:
    untracked: false
    expire_in: 30 days
    paths:
      - ./public

Test the artifact:
  stage: test
  image: alpine
  script:
  - grep -q "Gatsby" ./public/index.html

Test Web Site:
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve &
    - sleep 10
    - echo "curl http://localhost:9000 | grep -q "Gatsby" ./public/index.html"

Deploy staging: 
  stage: deploy staging
  environment:
    name: staging
    url: $STAGING_DOMAIN
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${url}"

Staging test:
  stage: staging test
  image: alpine
  environment:
    name: staging
    url: $STAGING_DOMAIN    
  script: 
    - apk add --no-cache curl
    - echo "curl http://${url}| grep 'Hi people'"
    - echo "curl http://${url}| grep $CI_COMMIT_SHORT_SHA" 


Deploy production: 
  stage: deploy production
  when: manual
  allow_failure: false
  only: 
    - master
  environment:
    name: production
    url: $PRODUCTION_DOMAIN
  script:
    - npm install --global surge
    - echo "surge --project ./public --domain ${url}"


Production test:
  stage: production test
  image: alpine
  only: 
    - master
  environment:
    name: production
    url: $PRODUCTION_DOMAIN  
  script: 
    - apk add --no-cache curl
    - echo "curl http://${PRODUCTION_DOMAIN}| grep 'Hi people'"
    - echo "curl http://${PRODUCTION_DOMAIN}| grep $CI_COMMIT_SHORT_SHA"     

```
