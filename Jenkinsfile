@Library('github.com/releaseworks/jenkinslib') _

pipeline {
  agent any

  environment {
    AWS_ACCOUNT_ID="168546287356"
    AWS_DEFAULT_REGION="us-east-1"
    CLUSTER_NAME="default1"
    SERVICE_NAME="nodejs-container-service"
    TASK_DEFINITION_NAME="first-run-task-definition"
    DESIRED_COUNT="1"
    IMAGE_REPO_NAME="express-test"
    IMAGE_TAG="${env.BUILD_ID}"
    REPOSITORY_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    registryCredential = "aws-admin-user"
    
    EXECUTION_ROLE_ARN = "arn:aws:iam::168546287356:role/ecsTaskExecutionRole"
    SHORT_COMMIT = "${GIT_COMMIT[0..7]}"
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
        def dockerImage = docker.build("$REPOSITORY_URI", ".")
        sh "docker build -t ${REPOSITORY_URI}:${IMAGE_TAG} ."

        // Push the Docker image to ECR
        //sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
      }
    }


    stage('Push Image to ECR') {
      steps {
       // sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
         // Push to ECR
        withElasticContainerRegistry {
                        docker.image("$REPOSITORY_URI").push("$SHORT_COMMIT")
                    }
        //sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}" 
      }
    }
    
    stage('Deploy to ECS') {
      steps {
            // prepare task definition file
                 sh """sed -e "s;%REPOSITORY_URI%;${REPOSITORY_URI};g" -e "s;%SHORT_COMMIT%;${SHORT_COMMIT};g" -e "s;%TASK_DEFINITION_NAME%;${TASK_DEFINITION_NAME};g" -e "s;%SERVICE_NAME%;${SERVICE_NAME};g" -e "s;%EXECUTION_ROLE_ARN%;${EXECUTION_ROLE_ARN};g" taskdef_template.json > taskdef_${SHORT_COMMIT}.json"""
                
                script {
                    // Register task definition
                    sh "aws ecs register-task-definition --output json --cli-input-json file://${WORKSPACE}/taskdef_${SHORT_COMMIT}.json > ${env.WORKSPACE}/temp.json"
                    def projects = readJSON file: "${env.WORKSPACE}/temp.json"
                    def TASK_REVISION = projects.taskDefinition.revision

                    // update service
                    sh "aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --task-definition ${TASK_DEFINITION_NAME}:${TASK_REVISION} --desired-count ${DESIRED_COUNT}"
                }
      }
    }
    
            stage('Remove docker image') {
            steps{
                // Remove images
                sh "docker rmi $REPOSITORY_URI"
            }
        }
    
    
  }
}
