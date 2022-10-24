pipeline {

  agent any

  stages {

    stage('Build') {

      when {

        expression {

          openshift.withCluster() {

            return !openshift.selector('bc', 'sample-app-jenkins').exists();

          }

        }

      }

      steps {

        script {

          openshift.withCluster() {

            openshift.newApp('redhat-openjdk18-openshift:1.1~https://github.com/kuldeepsingh99/openshift-jenkins-cicd.gitt')

          }

        }

      }

    }

  }

}
