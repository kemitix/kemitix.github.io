---
layout: post
title: "Template: Jenkinsfile"
date: 2018-02-24 22:09:00 +0000
last_updated: 2018-03-04 09:41:00 +0000
---
Checks out from Git, builds while updating snapshots, and if on the master branch, deploys.

Replace ${PROJECT} with the name of the project.

```jenkins
final String gitRepoUrl = 'git@github.com:kemitix/${PROJECT}.git'
final String mvn = "mvn --batch-mode --update-snapshots"

pipeline {
    agent any
    stages {
        stage('Prepare') {
            steps {
                git url: gitRepoUrl, branch: '**', credentialsId: 'github-kemitix'
            }
        }
        stage('no SNAPSHOT in master') {
            // checks that the pom version is not a snapshot when the current branch is master
            // TODO: also check for SNAPSHOT when is a pull request with master as the target branch
            when {
                expression {
                    (env.GIT_BRANCH == 'master') &&
                            (readMavenPom(file: 'pom.xml').version).contains("SNAPSHOT") }
            }
            steps {
                error("Build failed because SNAPSHOT version")
            }
        }
        stage('Build') {
            parallel {
                stage('Java 8') {
                    steps {
                        withMaven(maven: 'maven 3.5.2', jdk: 'JDK 1.8') {
                            sh "${mvn} clean install"
                        }
                    }
                }
                stage('Java 9') {
                    steps {
                        withMaven(maven: 'maven 3.5.2', jdk: 'JDK 9') {
                            sh "${mvn} clean install"
                        }
                    }
                }
            }
        }
        stage('Test Results') {
            steps {
                junit '**/target/surefire-reports/*.xml'
            }
        }
        stage('Archiving') {
            steps {
                archiveArtifacts '**/target/*.jar'
            }
        }
        stage('Coverage') {
            steps {
                jacoco(execPattern: '**/target/jacoco.exec')
            }
        }
        stage('Deploy') {
            when { expression { (env.GIT_BRANCH == 'master') } }
            steps {
                withMaven(maven: 'maven 3.5.2', jdk: 'JDK 1.8') {
                    sh "${mvn} deploy --activate-profiles release -DskipTests=true"
                }
            }
        }
    }
}
```
