pipeline {
  agent any 
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                echo "PATH = ${PATH}"
                echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    stage ('Check secrets') {
       steps {
         sh 'trufflehog3 https://github.com/rdkamble/sec.git -f json -o truffelhog_output.json || true'
       }
    }
     stage ('Software Composition Analysis') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'owasp-dc'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
       }
  
     stage ('Static Application Security Testing') {
	    steps {
            withSonarQubeEnv('Sonar-scanner') {
	            sh 'mvn sonar:sonar'
	        }
	    }
     }
      
      stage ('Deploy to Server Application') {
            steps {
                sshagent(['application-server']) {
                    sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/vulnweb/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@15.207.82.159:~'
                    sh 'ssh -o  StrictHostKeyChecking=no ubuntu@15.207.82.159 "nohup java -jar webgoat-server-v8.2.0-SNAPSHOT.jar --server.address=0.0.0.0 --server.port=8085 &"'
                }
            }     
       }
      
      
       stage ('Dynamic analysis') {
            steps {
          	    sshagent(['application-server']) {
                    sh 'ssh -o  StrictHostKeyChecking=no ubuntu@65.1.20.161 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://15.207.82.159:8085/WebGoat -x zap_report || true" '
                }      
            }       
        }
      
   }
}  
