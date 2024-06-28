pipeline {

  environment {
    registry = "milstein/nodeapp" // To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
    dockerImageName = ""
    dockerImage = ""
  }

  agent any

  stages {

    stage('Checkout Source') {
      steps {
        git branch: 'main', url: 'https://github.com/nerc-project/nodeapp.git'
      }
    }

    stage('Build Image') {
      steps{
        script {
          dockerImageName = "${registry}:${env.BUILD_NUMBER}"
          dockerImage = docker.build dockerImageName
        }
      }
    }

    stage('Pushing Image') {
      environment {
        registryCredential = 'dockerhublogin'
      }
      steps{
        script {
          docker.withRegistry('https://registry.hub.docker.com', registryCredential){
            dockerImage.push()
          }
        }
      }
    }

    stage('Docker Remove Image') {
      steps {
        sh "docker rmi -f ${dockerImageName}"
      }
    }

    stage('Deploying App to Kubernetes') {      
      steps {
        withKubeConfig([credentialsId: 'kubernetes']) {
          sh "sed -i 's/nodeapp:latest/nodeapp:${env.BUILD_NUMBER}/g' deploymentservice.yml"
          sh 'kubectl apply -f deploymentservice.yml'
        }
      }
    }
  }
}
