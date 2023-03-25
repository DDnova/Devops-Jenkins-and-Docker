pipeline {
  agent any
  environment {
        AWS_ACCOUNT_ID="168546287356"
        AWS_DEFAULT_REGION="us-east-1"
        CLUSTER_NAME="default"
        SERVICE_NAME="nodejs-container-service"
        TASK_DEFINITION_NAME="first-run-task-definition"
        DESIRED_COUNT="1"
        IMAGE_REPO_NAME="express-test"
        IMAGE_TAG="${env.BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        registryCredential = "aws-admin-user"
      }

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
         // def imageName = "my-app:${env.BUILD_NUMBER}"

          // Register the task definition
          sh "aws ecs register-task-definition --family $TASK_DEFINITION_NAME --container-definitions '[{\"name\":\"my-container\",\"image\":\"$IMAGE_REPO_NAME\",\"portMappings\":[{\"containerPort\":3000}]}]'"

          // Update the ECS service with the new task definition
          sh "aws ecs update-service --cluster $CLUSTER_NAME --service $SERVICE_NAME --task-definition $TASK_DEFINITION_NAME"
        }
      }
    }
  }
}
