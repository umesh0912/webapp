pipeline {
  agent any 
  environment {
    def imageLine = 'tomcat'
  }
  tools {
    maven 'Maven'
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
    
   stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/adarshreddy94/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
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
                sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@13.127.175.98:/opt/apache-tomcat-8.5.57/webapps/webapp.war'
              }      
           }       
    }
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@15.207.54.164 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://13.127.175.98:8080/webapp/" || true'
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
