pipeline {
    agent any
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

        stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat-server']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war root@127.0.0.1:/home/devsecops/apache-tomcat-8.5.38/webapps/'
              }      
           }
        }
    }
}
