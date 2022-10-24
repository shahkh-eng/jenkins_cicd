node('maven') {
  stage('Build') {
    git url: "https://github.com/kuldeepsingh99/openshift-jenkins-cicd.git"
    sh "mvn package"
    stash name:"jar", includes:"target/cart.jar"
  }
  stage('Build Image') {
    unstash name:"jar"
    sh "oc start-build cart --from-file=target/cart.jar --follow"
  }
  stage('Deploy') {
    openshiftDeploy depCfg: 'cart'
    openshiftVerifyDeployment depCfg: 'cart', replicaCount: 1, verifyReplicaCount: true
  }
 }
