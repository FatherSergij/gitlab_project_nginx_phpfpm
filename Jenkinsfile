pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: "5"))
    }
    agent any
    environment {
        IP_K8S="16.170.42.2"
        AWS_ACCOUNT_ID="728490037630"
        AWS_REGION="eu-north-1" 
        IMAGE_REPO_NAME="bigproject"
        BRANCH="${env.BRANCH_NAME}"
        //IMAGE_TAG="${env.BRANCH_NAME}-svc1-v${env.BUILD_NUMBER}"
        REPOSITORY_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        IMAGE_TAG="${GIT_COMMIT}"        
    }    
    
    stages {       

         stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                }
            }
        }   
    
         stage("Building image from branche nginx-phpfpm") {
            when { 
                allOf {
                    changeset "src/*"
                    anyOf {
                        not { triggeredBy cause: 'UserIdCause' }
                        branch 'develop'
                    }
                }
            }
            steps {
                script {
                       sh "docker build src/ -t ${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
            }
        } 

        
        stage("Pushing image to ECR nginx-phpfpm") {
            when { 
                allOf {
                    changeset "src/*"
                    anyOf {
                        not { triggeredBy cause: 'UserIdCause' }
                        branch 'develop'
                    }                    
                }
            }            
            steps {
                script {
                     sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"
                     sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
               }
            }
        }
        stage("Deploy on k8s from nginx-phpfpm") {
            when { 
                anyOf {
                    allOf {
                        branch 'develop'
                        changeset "src/*"
                    }
                    allOf {
                        triggeredBy cause: 'UserIdCause'
                        anyOf {
                            branch 'release'
                            branch 'master'
                        }
                    }
                }
            } 
            steps {
                script {
                    git url:"git@github.com:FatherSergij/project_lib_deploy.git",
                    branch:"${BRANCH}",
                    imageTag:"${IMAGE_TAG}",
                    //credentialsId:'jenkins',
                    load 'Jenkinsfile'
                }
            }
        }
    }
}
