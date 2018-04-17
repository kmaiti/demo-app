#!/usr/bin/env groovy

pipeline {

  agent any
  stages {
    stage('Build') {
      steps {
       configFileProvider([configFile(fileId: 'maven-settings-file', variable: 'MAVEN_SETTINGS')]) {
        withCredentials([usernamePassword(credentialsId: 'dis-functional', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {

             sh """
              mvn -version
              mvn -X -s $MAVEN_SETTINGS -Drevision=latest -U clean  deploy

              echo "STARTED: Docker Nexus Login"
              docker login localhost:5000 --username ${USERNAME} --password ${PASSWORD}

             echo "STARTED: Docker image build"
             docker build  -t localhost:5000/demo-app:latest .

             echo "STARTED: Docker images push"
             docker push localhost:5000/demo-app:latest
			 
			 echo "Remove image from local machine"
			 docker rmi localhost:5000/demo-app:latest
			 
			 
              """
			  }        
           }
       }
    }

    stage('Deploy') {
      steps {
	    script {
		  
          sh """
		   
           ssh ubuntu@10.0.0.140 bash -c "'
             
			sudo docker ps -a -q --filter ancestor=10.0.0.207:5000/demo-app:latest |xargs -r docker stop
           '"
        
		   
          """

        }
	   }	
    }
 }
}

