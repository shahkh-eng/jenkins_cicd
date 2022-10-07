@Library(['pipeline-toolbox','pipeline-framework'])
retriever: modernSCM(
  [
    $class: "GitSCMSource",
    remote: "https://github.com/kuldeepsingh99/openshift-jenkins-cicd.git"
  ]
)

appName = "hello-openshift-jenkins"


pipeline {
  // Use Jenkins Maven slave
  // Jenkins will dynamically provision this as OpenShift Pod
  // All the stages and steps of this Pipeline will be executed on this Pod
  // After Pipeline completes the Pod is killed so every run will have clean
  // workspace
  agent {
    label 'maven'
  }

  // Pipeline Stages start here
  // Requeres at least one stage
  stages {

    // Checkout source code
    // This is required as Pipeline code is originally checkedout to
    // Jenkins Master but this will also pull this same code to this slave
    stage('Git Checkout') {
      steps {
        // Turn off Git's SSL cert check, uncomment if needed
        // sh 'git config --global http.sslVerify false'
        checkout scm
      }
    }

    // Run Maven build, skipping tests
    stage('Build'){
      steps {
        sh "mvn -B clean install -DskipTests=true"
      }
    }

    // Build Container Image using the artifacts produced in previous stages
    stage('Build Container Image'){
      steps {
        // Copy the resulting artifacts into common directory
        sh """
          ls target/*
          rm -rf oc-build && mkdir -p oc-build/deployments
          for t in \$(echo "jar;war;ear" | tr ";" "\\n"); do
            cp -rfv ./target/*.\$t oc-build/deployments/ 2> /dev/null || echo "No \$t files"
          done
        """

        // Build container image using local Openshift cluster
        // Giving all the artifacts to OpenShift Binary Build
        // This places your artifacts into right location inside your S2I image
        // if the S2I image supports it.
        binaryBuild(pbuildConfigName: appName, buildFromPath: "oc-build")
      }
    }

  }
}
