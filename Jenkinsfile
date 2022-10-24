pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'

        sh 'mvn clean package'
        
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {
          openshift.withCluster() {
            openshift.withProject("kuldeepfr99-dev") {
                def buildConfigExists = openshift.selector("bc", "sample-app-jenkins").exists()

                if(!buildConfigExists){
                    openshift.newBuild("--name=sample-app-jenkins", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary")
                }

                openshift.selector("bc", "sample-app-jenkins").startBuild("--from-file=target/openshift-jenkins-cicd.jar", "--follow")

            }

          }
        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {
          openshift.withCluster() {
            openshift.withProject("kuldeepfr99-dev") {

              def deployment = openshift.selector("dc", "sample-app-jenkins")

              if(!deployment.exists()){
                openshift.newApp('sample-app-jenkins', "--as-deployment-config").narrow('svc').expose()
              }

              timeout(5) { 
                openshift.selector("dc", "sample-app-jenkins").related('pods').untilEach(1) {
                  return (it.object().status.phase == "Running")
                  }
                }

            }

          }
        }
      }
    }
  }
}
