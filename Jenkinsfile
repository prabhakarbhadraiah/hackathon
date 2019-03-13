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

	stage ('Source-Composition-Analysis') {
		steps {
			mvn 'org.owasp:dependency-check-maven:check -Ddependency-check-format=XML'
			step([$class: 'DependencyCheckPublisher', unstableTotalAll: '0'])
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
