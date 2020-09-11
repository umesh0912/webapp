pipeline {
  agent any 
  environment {
    def imageLine = 'adarshreddydocker/devsecops:test'
  }
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
    stage ('git-checkout') {
      steps {
        git 'https://github.com/adarshreddy94/webapp.git'
      }
    }
    
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json  https://github.com/adarshreddy94/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
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
                sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@3.236.209.93:/opt/apache-tomcat-8.5.57/webapps/webapp.war'
              }      
           }       
    }
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@3.236.102.28 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://3.236.209.93:8080/webapp/" || true'
        }
      }
    }
   
    stage ('Anchore-Container-scanner') {
        steps {
            writeFile file: 'anchore_images', text: imageLine
            anchore name: 'anchore_images'
        }
    }
    
  }
}
