---
layout: post
title: "Template: Jenkinsfile"
date: 2018-02-24 22:09:00 +0000
last_updated: 2018-02-25 15:50:00 +0000
---
Checks out from Git, builds while updating snapshots, and if on the master branch, deploys.

Replace ${PROJECT} with the name of the project.

```jenkins
pipeline {
    agent any
    stages {
        stage('Prepare') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '**']],
                    extensions: [[$class: 'CleanBeforeCheckout']],
                    userRemoteConfigs: [[
                        credentialsId: 'github-kemitix', 
                        url: 'git@github.com:kemitix/${PROJECT}.git']]
                ])
            }
        }
        stage('Build') {
            steps {
                sh './mvnw -B -U clean install'
            }
        }
        stage('Reporting') {
            steps {
                junit '**/target/surefire-reports/*.xml'
                archiveArtifacts '**/target/*.jar'
            }
        }
        stage('Deploy') {
            when {
                expression {
                    env.GIT_BRANCH == 'master'
                }
            }
            steps {
                sh './mvnw -B -P release deploy'
            }
        }
    }
}
```
