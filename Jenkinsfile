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
        sh 'rm -rf trufflehog || true'
        sh 'docker run -v "$PWD:/pwd" ghcr.io/trufflesecurity/trufflehog:latest github --repo https://github.com/devops-CloudComputing/DevSecOps_CICD_Jenkins_Pipeline.git --debug --json > trufflehog'
        sh 'cat trufflehog'
    }
}
   stage ('Source Composition Analysis') {
    steps {
        sh 'rm owasp* || true'
        sh ' wget "https://raw.githubusercontent.com/devops-CloudComputing/DevSecOps_CICD_Jenkins_Pipeline/master/owasp-dependency-check.sh" '
        sh 'chmod +x owasp-dependency-check.sh'
        sh 'bash owasp-dependency-check.sh'
        sh 'cat /var/lib/jenkins/workspace/webapp-cicd-pipeline/odc-reports/dependency-check-report.xml'
        
      }
    }


   stage ('SAST') {
    steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
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
  
    stage ('DAST') {
     steps {
        sshagent(['zap-baseline']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@54.166.177.99 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://54.173.72.247:8080/webapp/" || true'
        }
      }
    }
  }
  
}
