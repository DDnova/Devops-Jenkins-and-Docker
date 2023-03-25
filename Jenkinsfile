pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        // Checkout the code from the Git repository
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/DDnova/express.git']]])

        // Build the Docker image
        script {
          def imageName = "my-express-app:${env.BUILD_NUMBER}"
          def containerName = "my-express-container"

          sh "docker build -t $imageName ."
        }
      }
    }

    stage('Deploy') {
      steps {
        // Stop and remove any existing container with the same name
        script {
          def containerName = "my-express-container"

          sh "docker stop $containerName || true"
          sh "docker rm -f $containerName || true"
        }

        // Run a new container from the built image
        script {
          def imageName = "my-express-app:${env.BUILD_NUMBER}"
          def containerName = "my-express-container"

          sh "docker run -p 3000:3000 -d --name $containerName $imageName"
        }
      }
    }
  }
}
