pipeline {
  agent any
  environment {
    def imageLine = 'adarshreddydocker/devsecops:newwebapp'
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
        git 'https://github.com/umesh0912/webapp.git'
      }
    }

    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/umesh0912/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/workspace/devsecops_kiuwan_integration/odc-reports/dependency-check-report.xml'
        
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
  
    stage ('Kiuwan') {
      steps {
       kiuwan connectionProfileUuid: 'n2i0-2KgZ'
    }
  }  
    
    stage ('Deploy-To-Tomcat') {
            steps {
             sshagent(['tomcat']) {
                  sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@65.1.229.50:/opt/apache-tomcat-9.0.44/webapps/WebApp.war'
           }       
         }
    }
    
    stage ('DAST') {
        steps {
         sh "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://65.1.229.50:8080/WebApp/" 
        }
    }   
    
  }
}
