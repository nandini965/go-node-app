pipeline {
  agent any

   stages {
   stage('package manager') {
   steps {
   sh 'npm install'
   }
   }
  stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://34.201.116.83:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }


    stage('docker build and run') {
      environment {
        DOCKER_IMAGE = "nandini965/go-node-app:${BUILD_NUMBER}"
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
          script {
            sh '''
              docker build -t ${DOCKER_IMAGE} .
              echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
              docker push ${DOCKER_IMAGE}
              docker logout
            '''
          }
        }
      }
    }

    stage('Update Deployment File') {
         environment {
           GIT_REPO_NAME = "go-web-app-practice"
           GIT_USER_NAME = "nandini965"
         }
         steps {
           withCredentials([string(credentialsId: 'github-cred', variable: 'GITHUB_TOKEN')]) {
             sh '''
               git config user.email "nandhinigoud965@gmail.com"
               git config user.name "nandini965"
               sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" k8s/manifests/deployment.yml
               git add k8s/manifests/deployment.yml
               git commit -m "Update deployment image to version ${BUILD_NUMBER}" || echo "No changes to commit"
               git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
             '''
           }
         }
       }

     }
   }