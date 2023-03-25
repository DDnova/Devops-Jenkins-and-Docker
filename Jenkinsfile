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
          def imageName = "${IMAGE_REPO_NAME}:${env.BUILD_ID}"

          sh "docker build -t $imageName ."
        }
      }
    }

    stage('Push') {
      steps {
        script {
          withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: "${registryCredential}", secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
            sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"

            sh "docker tag ${IMAGE_REPO_NAME}:${env.BUILD_ID} ${REPOSITORY_URI}:${IMAGE_TAG}"

            sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          def taskDefinition = [:]
          taskDefinition.family = "${TASK_DEFINITION_NAME}"
          taskDefinition.containerDefinitions = [
            [
              name: "${IMAGE_REPO_NAME}",
              image: "${REPOSITORY_URI}:${IMAGE_TAG}",
              portMappings: [
                [
                  containerPort: 3000,
                  hostPort: 3000
                ]
              ],
              essential: true
            ]
          ]

          sh "echo '${taskDefinition}' > taskdefinition.json"

          sh "aws ecs register-task-definition --cli-input-json file://taskdefinition.json"

          sh "aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --force-new-deployment --task-definition ${TASK_DEFINITION_NAME}:${BUILD_NUMBER} --desired-count ${DESIRED_COUNT}"
        }
      }
    }
  }
}
