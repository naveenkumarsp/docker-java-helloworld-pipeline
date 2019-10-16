pipeline {
   agent any

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
     
     SERVICE_NAME = "docker-java-helloworld-pipeline"
     IMAGE_NAME = "ci-pipeline-demo-${jenkins_username}"
     REPOSITORY_TAG="${DOCKERHUB_URL}/${IMAGE_NAME}:${BUILD_ID}"
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }
      stage('Update user references') {
         steps {
            sh 'cat ./src/main/webapp/index.jsp'
            sh """sed -i 's+Admin+'"${jenkins_username}"'+' ./src/main/webapp/index.jsp"""
            sh 'cat ./src/main/webapp/index.jsp'
            sh 'ls -all'
            sh """sed -i 's+jenkins_username+'"${jenkins_username}"'+' deploy.yaml"""
            sh """sed -i 's+REPOSITORY_TAG+'"${REPOSITORY_TAG}"'+' deploy.yaml"""
            sh """sed -i 's+jenkins_username+'"${jenkins_username}"'+' ingress-service.yaml"""
         }
      }
      stage('Build') {
         steps {
            sh '''mvn clean install package'''
         }
      }

      stage('SonarQube') {
         steps {
            sh '${sonarcli}'
         }
      }
      stage('Build Image') {
         steps {
           sh 'scp -r ${WORKSPACE} jenkins@${DOCKER_HOST_IP}:/home/jenkins/docker/${BUILD_ID}'
           sh 'ssh jenkins@${DOCKER_HOST_IP} docker image build -t ${REPOSITORY_TAG} /home/jenkins/docker/${BUILD_ID}'
           sh 'ssh jenkins@${DOCKER_HOST_IP} docker image ls'
           sh 'ssh jenkins@${DOCKER_HOST_IP} rm -rf /home/jenkins/docker/${BUILD_ID}'
         }
      }
      stage('Push Image to repo') {
          steps {
           sh 'ssh jenkins@${DOCKER_HOST_IP} docker push ${REPOSITORY_TAG}'
          }
      }
      stage('Deploy the application') {
          steps {
            sh 'cat deploy.yaml'
            sh 'kubectl apply -f deploy.yaml'
            sh 'cat ingress-service.yaml'
            sh 'kubectl apply -f ingress-service.yaml'
          }
      }
   }
}
