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
		    ssh -T 10.0.0.140 << EOSSH
              # setup variables

              IMAGE_VERSION_TO_BE_DEPLOYED=latest
              APP=demo-app
              NEXUS_REGISTRY=10.0.0.207:5000
              DOCKER_USERNAME=dis-functional
              DOCKER_PASSWORD=dis-functional
              LOCAL_IMAGE_IDS=`sudo docker images|grep $APP|awk '{print $1}'`
              
              # Check running container and stop it
              RUNNING_CONTAINER=`sudo docker ps|grep -i $APP|head -1|awk '{print $1}'`
              
              if [ -z $RUNNING_CONTAINER ]; then
                # container is not running, remove all images of the app.
                for i in `sudo docker ps -a|grep $APP|awk '{print $1}'`; do sudo docker rm $i; done
                echo "$LOCAL_IMAGE_IDS"|while read id; do sudo docker rmi -f $id ; done
              else
                #container is running
                sudo docker stop $RUNNING_CONTAINER
                for i in `sudo docker ps -a|grep $APP|awk '{print $1}'`; do sudo docker rm $i; done
                echo "$LOCAL_IMAGE_IDS"|while read id; do sudo docker rmi -f $id ; done
              fi
              
              # Now start fresh container with fresh image
              
              NEW_IMAGE="$NEXUS_REGISTRY/$APP:$IMAGE_VERSION_TO_BE_DEPLOYED"
              # start container in daemon mode
              sudo docker login $NEXUS_REGISTRY -u="$DOCKER_USERNAME" -p="$DOCKER_PASSWORD"
              sudo docker run -itd  --name $APP -p 8000:8000 $NEW_IMAGE
              
              # check if it is running or not
              sudo docker ps |grep $APP
           EOSSH
          """
        }
    }

 }
}

