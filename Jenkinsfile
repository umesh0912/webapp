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
    stage ('git-checkout') {
      steps {
        git 'https://github.com/adarshreddy24/webapp.git'
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
           deploy adapters: [tomcat9(credentialsId: 'a5fb49e1-0231-47e3-b410-11025f016b03', path: '', url: 'http://65.1.229.50:8080/')], contextPath: '/opt/apache-tomcat-9.0.44/webapps', war: '/*.war'    
           }       
    }
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@65.0.175.77 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://65.1.229.50:8080/webapp/" || true'
        }
      }
    }
    
  }
}
