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
    
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json  https://github.com/adarshreddy24/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/adarshreddy94/webapp/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/workspace/webapp_Pipeline/odc-reports/dependency-check-report.xml'
        
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
            sh "curl -v -u devsecops:^^B45$$e!77 -T /var/lib/jenkins/workspace/webapp_Pipeline/target 'http:// 65.1.229.50:8080//manager/text/deploy?path=/pipeline_webapp'"
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
