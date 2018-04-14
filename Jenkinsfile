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
              mvn -X -s $MAVEN_SETTINGS -Drevision=latest -U -X clean  deploy

              echo "STARTED: Docker Nexus Login"
              docker login 13.127.248.170:5000 --username ${USERNAME} --password ${PASSWORD}

             echo "STARTED: Docker image build"
             docker build  -t 13.127.248.170:5000/demo-app:latest .

             echo "STARTED: Docker images push"
             docker push 13.127.248.170:5000/demo-app:latest
              """
			  }        
           }
       }
    }




    stage('Deploy') {
      steps {
          sh """
           echo test
          """
        }
    }

 }
}

