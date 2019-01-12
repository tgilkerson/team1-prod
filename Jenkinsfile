import java.text.SimpleDateFormat

pipeline {
  options {
    buildDiscarder logRotator(numToKeepStr: '5')
    disableConcurrentBuilds()
  }
  agent {
    kubernetes {
      cloud "kubernetes"
      label "team1-prod"
      serviceAccount "build"
      yamlFile "KubernetesPod.yaml"
    }
  }
  environment {
    cmAddr = "cm-chartmuseum.charts:8080"
  }
  stages {
    stage("deploy") {
      when {
        branch "master"
      }
      steps {
        container("helm") {
          withCredentials([usernamePassword(credentialsId: "chartmuseum", usernameVariable: "USER", passwordVariable: "PASS")]) {
            sh "helm repo add chartmuseum http://$USER:$PASS@${cmAddr}"
            sh "helm repo update"
            sh "helm dependency update helm"
            sh "helm upgrade -i team1-prod helm --namespace team1-prod --force"
          }
        }
      }
    }
    stage("test") {
      when {
        branch "master"
      }
      steps {
        echo "Testing..."
      }
      post {
        failure {
          container("helm") {
            sh "helm rollback prod 0"
          }
        }
      }
    }
  }
}
