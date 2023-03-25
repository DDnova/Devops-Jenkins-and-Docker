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
    REPOSITORY_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    registryCredential = "aws-admin-user"
  }

  stages {
    stage('Logging into AWS ECR') {
        steps {
          script {
            sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
              }
          }
       }
    stage('Build') {
      steps {
        // Checkout the code from the Git repository
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/DDnova/express.git']]])


        // Build the Docker image
        sh "docker build -t ${REPOSITORY_URI}:${IMAGE_TAG} ."

        // Push the Docker image to ECR
        //sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
      }
    }


    stage('Push Image to ECR') {
      steps {
        sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
        sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}" 
      }
    }
    
    stage('Deploy in ECS') {
       steps {
          withCredentials([string(credentialsId: 'AWS_EXECUTION_ROL_SECRET', variable: 'AWS_ECS_EXECUTION_ROL'),string(credentialsId: 'AWS_REPOSITORY_URL_SECRET', variable: 'AWS_ECR_URL')]) {
      script {
        updateContainerDefinitionJsonWithImageVersion()
        sh("/usr/local/bin/aws ecs register-task-definition --region ${AWS_DEFAULT_REGION} --family ${TASK_DEFINITION_NAME}")
        def taskRevision = sh(script: "/usr/local/bin/aws ecs describe-task-definition --task-definition ${AWS_ECS_TASK_DEFINITION} | egrep \"revision\" | tr \"/\" \" \" | awk '{print \$2}' | sed 's/\"\$//'", returnStdout: true)
        sh("/usr/local/bin/aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --task-definition ${TASK_DEFINITION_NAME}:${taskRevision}")
              }
           }
        }
      
    }
    
    
    
  }
}
