# gitlab

## CI/CD
* Basic CI/CD workflow with Gitlab CI
* What is CI / CD?

### Continious Integration
* Code -> CI pipleine ( Build -> Code Quality -> Test -> Package)

### Continious Delivery
* Continious Delivery= previous stages you have create a package, auto install this package in an ENV decide if a specific package should go in PRD
* CD pipeline ( Review / Test -> Stagging -> questions? -> Production )

## Sample
* https://gitlab.com/zzdanilou/car-asembly-line
* Create file .gitlab-ci.yml
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
    - curl http://localhost:8000 | grep -q "Gatsby" ./public/index.html

* Deployment using surge.sh
# Use surge.sh for the deployment
npm install --global surge

Need to create an account in surge
surge token

Settings => CI/CD => Variables => Add Variable
=> Key / Value

Protect Variable disable or the var is only available for a branch
Create
SURGE_LOGIN  Disable protect var, Disable Mask var
SURGE_TOKEN Disable Protect var, Enable Mask var (secret)

```
