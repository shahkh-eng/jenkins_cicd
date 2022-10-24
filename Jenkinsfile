#! /usr/bin/env groovy

pipeline {

  agent {
      label 'maven'
  }
  stages {
    stage('Build App') {
      steps {
        sh "mvn install"
      }
    }
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector("bc", "sample-app-jenkins").exists();
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newBuild("--name=sample-app-jenkins", "--image-stream=redhat-openjdk18-openshift:1.1", "--binary")
          }
        }
      }
    }
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.selector("bc", "sample-app-jenkins").startBuild("--from-file=target/sample-app-jenkins.jar", "--wait")
          }
        }
      }
    }
    stage('Promote to DEV') {
      steps {
        script {
          openshift.withCluster() {
            openshift.tag("sample-app-jenkins:latest", "sample-app-jenkins:dev")
          }
        }
      }
    }
    stage('Create DEV') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector('dc', 'sample-app-jenkins').exists()
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newApp("sample-app-jenkins:latest", "--name=sample-app-jenkins").narrow('svc').expose()
          }
        }
      }
    }
  }
}
