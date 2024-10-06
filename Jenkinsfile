pipeline {
  environment {
    dockerimagename = "baldinlex/diplom-app"
    dockerImage = ""
    
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
        git branch: 'main', url: 'https://github.com/baldin-lex/diplom-app-test.git'
      }
    }
    stage('Checkout tag') {
      steps{
        script {
          sh 'git fetch'
          gitTag=sh(returnStdout:  true, script: "git tag --sort=-creatordate | head -n 1").trim()
          echo "gitTag output: ${gitTag}"
        }
      }
    }
    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }
    stage('Pushing Image:tags') {
      environment {
               registryCredential = 'dockerhub-credentials'
           }
      steps{
        script {
          docker.withRegistry( 'https://index.docker.io/v1/', registryCredential ) {
            dockerImage.push("${gitTag}")
          }
        }
      }
    }
    stage('sed env') {
      environment {
              envTag = ("${gitTag}")
           }    
      steps{
        script {
          sh "sed -i \'18,22 s/gitTag/\'$envTag\'/g\' myapp-deploy.yml"
          sh 'cat myapp-deploy.yml'
        }
      }
    }
    stage('Deploying myapp-deploy to Kubernetes') {
      steps {
        script {
          withKubeConfig(credentialsId: 'k8s-credentials', serverUrl: 'https://84.201.180.234:6443') {
            sh 'kubectl apply -f "myapp-deploy.yml"'
          }
        }
      }
    }
  }    
} 
