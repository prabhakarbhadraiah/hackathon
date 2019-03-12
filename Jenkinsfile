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
                archiveArtifacts 'target/*.war'
            }
        }
        
        stage ('Deploy') {
            steps {
                sshagent(['host']) {
                       sh 'scp -o StrictHostKeyChecking=no target/*.war root@172.17.0.1:/home/devsecops/apache-tomcat-8.5.38/webapps/'
                }
            }
        }
    }
}
