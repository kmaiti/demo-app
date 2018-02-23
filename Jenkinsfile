#!/usr/bin/env groovy

pipeline {

  agent any
  stages {
    stage('Build') {
     agent {
        label "docker"
      }
      steps {
            configFileProvider([configFile(fileId: 'maven-settings-file', variable: 'MAVEN_SETTINGS')]) {
          container('docker') {
                     withCredentials([usernamePassword(credentialsId: 'nexus-functional-user', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {

             sh """
              mvn -version
              mvn -X -s $MAVEN_SETTINGS -Drevision=latest -U -X clean  deploy

              echo "STARTED: Docker Nexus Login"
              docker login nexus.tools.aws.cps.vodafone.com --username ${USERNAME} --password ${PASSWORD}

             echo "STARTED: Docker image build"
             docker build  -t nexus.tools.aws.cps.vodafone.com/demo-app:latest .

             echo "STARTED: Docker images push"
             docker push nexus.tools.aws.cps.vodafone.com/demo-app:latest
              """

                   }
        }
           }
      }
    }




    stage('Deploy') {
      agent {
        label "kubectl"
      }
      steps {
        container('kubectl') {
          sh 'which kubectl'
          sh """
           kubectl --namespace=test apply -f k8s-manifests
          """
        }
       }

    }

 }
}

