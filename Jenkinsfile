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
		     sh 'wget https://raw.githubusercontent.com/devopssecure/webapp/master/owasp-dependency-check.sh'	
		     sh 'bash owasp-dependency-check.sh'
		     sh ''' python upload-results.py --host 3.81.3.77:8000 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 1 --result_file /root/OWASP-Dependency-Check/reports/dependency-check-report.xml --username admin --scanner "Dependency Check Scan" ''' 
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
