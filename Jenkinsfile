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
	    
	stage ('Source-Composition-Analysis') {
		steps {
		     sh 'bash owasp-dependency-check.sh'
		     sh ''' python upload-results.py --host 127.0.0.1:8000 --api_key 374d66dad8b2b10a54437d04e5fa83819a10b752 --engagement_id 1 --result_file /root/OWASP-Dependency-Check/reports/dependency-check-report.xml --username admin --scanner "Dependency Check Scan" ''' 
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
