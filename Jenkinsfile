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
			 
			 echo "testing for Mayank"
              """
			  }        
           }
       }
    }




    stage('Deploy') {
      steps {
          sh """
		  echo "ssh to machine where image will be deplyed, stop running container, start it with new image.IP will be nexus registry IP"
           echo test
          """
        }
    }

 }
}

