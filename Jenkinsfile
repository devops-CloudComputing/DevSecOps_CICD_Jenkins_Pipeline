pipeline {
  agent any
  tools {
    maven 'maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
          echo "PATH = ${PATH}"
          echo "M2_HOME = ${M2_HOME}"
        ''' 
      }
    }
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm -rf trufflehog || true '
        sh ' docker run -it -v "$PWD:/pwd" ghcr.io/trufflesecurity/trufflehog:latest github --repo https://github.com/devops-CloudComputing/DevSecOps_CICD_Jenkins_Pipeline.git --debug --json > trufflehog '
        sh 'cat trufflehog'     
      
      }
    }

    stage ('Build') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage ('Deploy-To-Tomcat') {
      steps {
        sshagent(['tomcat']) {
          sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@172.31.49.31:/home/ubuntu/apache-tomcat-9.0.82/webapps/webapp.war'
        }
      }
    }
  }
}
