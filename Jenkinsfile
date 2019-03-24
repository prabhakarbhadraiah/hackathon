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
	    
	    stage ('Check-Git-Secrets') {
		    steps {
	        sh 'rm trufflehog || true'
		sh 'docker pull gesellix/trufflehog'
		sh 'docker run -t gesellix/trufflehog --json https://github.com/devopssecure/webapp.git > trufflehog'
		sh 'cat trufflehog'
	    }
	    }
	    

	  stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        }    
	    
	stage ('Source-Composition-Analysis') {
		steps {
		     sh 'rm owasp-* || true'
		     sh 'wget https://raw.githubusercontent.com/devopssecure/webapp/master/owasp-dependency-check.sh'	
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
       stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.war ubuntu@54.86.226.84:/home/ubuntu/prod/apache-tomcat-8.5.39/webapps/webapp.war'
              }      
           }       
    }
	 
	    stage ('Port Scan') {
		    steps {
			sh 'rm nmap* || true'
			sh 'docker run --rm -v "$(pwd)":/data uzyexe/nmap -sS -sV -oX nmap 54.86.226.84'
			sh 'cat nmap'
		    }
	    }
	    
	    stage ('DAST') {
		  
		    	steps {
			    sshagent(['zap']) {
				    sh 'ssh -o StrictHostKeyChecking=no ubuntu@3.85.77.1 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://54.86.226.84:8080/webapp/" || true'
			    }
			}
		}    
	
	    stage ('Nikto Scan') {
		    steps {
			sh 'rm nikto-output.xml || true'
			sh 'docker pull secfigo/nikto:latest'
			sh 'docker run --user $(id -u):$(id -g) --rm -v $(pwd):/report -i secfigo/nikto:latest -h 54.86.226.84 -p 8080 -output /report/nikto-output.xml'
			sh 'cat nikto-output.xml'   
		    }
	    }
	    
	    stage ('SSL Checks') {
		    steps {
			sh 'pip install sslyze==1.4.2'
			sh 'python -m sslyze --regular 54.86.226.84:8080 --json_out sslyze-output.json'
			sh 'cat sslyze-output.json'
		    }
	    }
	    
	    stage ('Upload Reports to Defect Dojo') {
		    steps {
			sh 'pip install requests'
			sh 'wget https://raw.githubusercontent.com/devopssecure/webapp/master/upload-results.py'
			sh 'chmod +x upload-results.py'
			sh 'python upload-results.py --host 3.81.3.77:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file trufflehog --username admin --scanner "SSL Labs Scan"'
			sh 'python upload-results.py --host 3.81.3.77:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml --username admin --scanner "Dependency Check Scan"'
			sh 'python upload-results.py --host 3.81.3.77:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file nmap --username admin --scanner "Nmap Scan"'
			sh 'python upload-results.py --host 3.81.3.77:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file sslyze-output.json --username admin --scanner "SSL Labs Scan"'
			sh 'python upload-results.py --host 3.81.3.77:80 --api_key 66879c160803596f132aff025fee9a170366f615 --engagement_id 4 --result_file nikto-output.xml --username admin'
			    
		    }
	    }
	    
	
    }
}
