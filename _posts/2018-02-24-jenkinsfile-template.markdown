---
layout: post
title: "Template: Jenkinsfile"
date: 2018-02-24 22:09:00 +0000
last_updated: 2018-03-09 23:21:00 +0000
---
Features:

* Records all environment variables
* Aborts if attempting to build a SNAPSHOT version on either the `master` branch or a pull request onto the `master` branch
* Perform Static Code Analysis with checkstyle and pmd if there are Java source files - will fail if plugins are not available
* Performs a test build/install with Java 9 to validate compatibility
* Performs build/install with Java 8 for deploy candidate
* Records test results and code coverage in Jenkins if tests were run by surefire/failsafe
* Publishes code coverage to Codacy if tests were run by surefire/failsafe - see helpers below
* Archives any jar file created
* Deploy if on master branch and is attached to a remote git repo (i.e. not a local file:/ repo).

```jenkins
final String mvn = "mvn --batch-mode --update-snapshots"

pipeline {
    agent any
    stages {
        stage('Environment') {
            steps {
                sh 'set'
            }
        }
        stage('no SNAPSHOT in master') {
            // checks that the pom version is not a snapshot when the current or target branch is master
            when {
                expression {
                    (env.GIT_BRANCH == 'master' || env.CHANGE_TARGET == 'master') &&
                            (readMavenPom(file: 'pom.xml').version).contains("SNAPSHOT")
                }
            }
            steps {
                error("Build failed because SNAPSHOT version")
            }
        }
        stage('Static Code Analysis') {
            when { expression { findFiles(glob: '**/src/main/java/*.java').length > 0 } }
            steps {
                withMaven(maven: 'maven 3.5.2', jdk: 'JDK 1.8') {
                    sh "${mvn} compile checkstyle:checkstyle pmd:pmd"
                }
                pmd canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''
            }
        }
        stage('Build Java 9') {
            steps {
                withMaven(maven: 'maven 3.5.2', jdk: 'JDK 9') {
                    sh "${mvn} clean install"
                }
            }
        }
        stage('Build Java 8') {
            steps {
                withMaven(maven: 'maven 3.5.2', jdk: 'JDK 1.8') {
                    sh "${mvn} clean install"
                }
            }
        }
        stage('Test Results') {
            when { expression { findFiles(glob: '**/target/surefire-reports/*.xml').length > 0 } }
            steps {
                junit '**/target/surefire-reports/*.xml'
                jacoco exclusionPattern: '**/*{Test|IT|Main|Application|Immutable}.class'
                withMaven(maven: 'maven 3.5.2', jdk: 'JDK 1.8') {
                    sh "${mvn} com.gavinmogan:codacy-maven-plugin:coverage " +
                            "-DcoverageReportFile=target/site/jacoco/jacoco.xml " +
                            "-DprojectToken=`$JENKINS_HOME/codacy/token` " +
                            "-DapiToken=`$JENKINS_HOME/codacy/apitoken` " +
                            "-Dcommit=`git rev-parse HEAD`"
                }
            }
        }
        stage('Archiving') {
            when { expression { findFiles(glob: '**/target/*.jar').length > 0 } }
            steps {
                archiveArtifacts '**/target/*.jar'
            }
        }
        stage('Deploy') {
            when { expression { (env.GIT_BRANCH == 'master' && env.GIT_URL.startsWith('https://')) } }
            steps {
                withMaven(maven: 'maven 3.5.2', jdk: 'JDK 1.8') {
                    sh "${mvn} deploy --activate-profiles release -DskipTests=true"
                }
            }
        }
    }
}
```

## Codacy

Publishing to Codacy requires having a secret API Token and a Repo Token. Neither of these should be added to the Jenkinsfile. Instead the two scripts here are interpolated into the maven command line that invokes the `codacy-maven-plugin` in the *Test Results* stage.

Requirements: `xpath` 
 
 > `sudo apt install libxml-xpath-perl`

### token

```bash
#!/usr/bin/env bash

TOKENS_FILE=`dirname $0`/tokens
ARTIFACT_ID=`xpath -q -e "/project/artifactId/text()" < pom.xml`

grep "^${ARTIFACT_ID}=" ${TOKENS_FILE}|cut -d= -f2
``` 

### tokens

Provides the repo token by searching for it in the text file `tokens`. e.g.
```none
repo1=repo1key
repo2=repo2key
```
These would match against the `artifactId` of the root module `pom.xml`.

### apitoken

Simply echos the apikey to STDOUT.

```bash
#!/usr/bin/env bash

echo "xxxXXXxxxXXXxxxXXXxxx"
```
