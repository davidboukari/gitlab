# gitlab

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
