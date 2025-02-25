# Scenario-2
------------

[![Build Status](https://travis-ci.org/kottapar/node-mongo-crud.svg?branch=master)](https://travis-ci.org/kottapar/node-mongo-crud)

The pipeline for this scenario is built using TravisCI and is available [here](https://travis-ci.org/kottapar/node-mongo-crud)

I've forked an existing project, made changes and created the pipeline as per our requirements.

VM Pre-requisites
-----------------

We would require a VM with Docker installed to get started. This is created and tested on a `Ubuntu 18.04` VM.

```
root@rabuntu:~/node-mongo-crud# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.3 LTS
Release:        18.04
Codename:       bionic

root@rabuntu:~/node-mongo-crud# docker --version
Docker version 19.03.1, build 74b1e89

```

Run `git clone https://github.com/kottapar/node-mongo-crud.git` to clone this repo.

As this is a project for demonstration, the example.env file is saved. Run `mv example.env .env` and edit the parameters if required.

For the Travis pipeline in production we've encrypted the .env file using the Travis gem which can be installed using `gem install travis -v 1.8.10 --no-rdoc --no-ri`. More info can be found [here](https://docs.travis-ci.com/user/encrypting-files/)

Build the docker image
----------------------

Run `docker-compose -f docker/docker-compose.yml build` to build the required images. You should see an output similar to the one below:

```
root@rabuntu:~/ravi/node-mongo-crud# docker-compose -f docker/docker-compose.yml build
database uses an image, skipping
Building backend
Step 1/11 : FROM node:10.12.0-alpine

**snip**

Successfully built 718886b536ea
Successfully tagged docker_backend:latest
root@rabuntu:~/ravi/node-mongo-crud#
root@rabuntu:~/ravi/node-mongo-crud# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
docker_backend        latest              718886b536ea        48 seconds ago      77.1MB
```

Run the unit tests on the docker image
--------------------------------------

If you'd like to run the tests manually run `docker-compose -f docker/docker-compose.test.yml up`. You should see an output similar to the one below:

```
root@rabuntu:~/ravi/node-mongo-crud# docker-compose -f docker/docker-compose.test.yml up
Creating docker_database-test_1 ...
Creating docker_backend-test_1 ...
Creating docker_database-test_1
Creating docker_backend-test_1 ... done
Attaching to docker_database-test_1, docker_backend-test_1

** snip **

backend-test_1   | Host database-test:27017 is now available
backend-test_1   |
backend-test_1   |
backend-test_1   | Listening on port 8090
backend-test_1   |   API Integration Test
backend-test_1   | TAP version 13
backend-test_1   | # /api/documents
backend-test_1   | ok 1 Created a new document successfully, test passed!
backend-test_1   | # /api/documents
backend-test_1   | ok 2 Got all documents successfully, test passed!
backend-test_1   | # /api/documents/:id
backend-test_1   | ok 3 Got a specific document successfully, test passed!
backend-test_1   | # /api/documents/:id
backend-test_1   | ok 4 Edited a document successfully, test passed!
backend-test_1   | # /api/documents/:id
backend-test_1   | ok 5 Deleted a specific document successfully, test passed!
backend-test_1   |     ✓ Runs all tests (294ms)
backend-test_1   |
backend-test_1   |
backend-test_1   |   1 passing (430ms)
backend-test_1   |
backend-test_1   |
backend-test_1   | 1..5
backend-test_1   | # tests 5
backend-test_1   | # pass  5
backend-test_1   |
backend-test_1   | # ok
backend-test_1   |
docker_backend-test_1 exited with code 0
```

Press Ctrl+c to exit and terminate the containers. 

TravisCI pipeline
-----------------

The stages of the pipeline are described below. To view the pipeline for this project you can click [here](https://travis-ci.org/kottapar/node-mongo-crud)

You can click on the job to view more details in verbose. However if you'd like to see the pipeline in action you can fork this project, register on TravisCI and then run the pipeline.

code scan using NodeJSScan
--------------------------
The open-source project [nodejssscan](https://github.com/ajinabraham/NodeJsScan) is used to perform a static security code scan of the js files in the project.

build the test docker image
---------------------------
A docker image is built from the production Dockerfile. It is used as a starting point to build a test docker image to perform the unit tests and scan the image for vulnerabilities.

The open-source project [trivy](https://github.com/knqyf263/trivy) is being used to perform the vulnerability scan of the image. It detects vulnerabilities of OS packages (Alpine, RHEL, CentOS, etc.) and application dependencies (Bundler, Composer, npm, yarn etc.)

Run the tests
-------------
This stage performs the unit and integration tests via `npm test`

After that trivy is ran and is configured to exit with a non-zero code if any critical vulnerabilities are detected in the scan.

push the image to private registry
----------------------------------
The image is then tagged and pushed to the repo. The docker credentials were saved as DOCKER_PASS and DOCKER_USER in TravisCI.
