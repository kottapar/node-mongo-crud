# node-mongo-crud

[![Build Status](https://travis-ci.org/kottapar/node-mongo-crud.svg?branch=master)](https://travis-ci.org/kottapar/node-mongo-crud)

A CRUD API built with MongoDB and Express. This project is forked to help with the below TravisCI pipeline.

The intention is to build a pipeline for a NodeJS web app that performs the unit tests and a security scan of the code before building the Docker image and pushing it a repo.

The stages are listed below

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
The image is then tagged and pushed to the repo. 
