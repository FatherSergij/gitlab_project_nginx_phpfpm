pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: "5"))
    }    
    agent any

    libraries {
         lib('lib-for-project')
    }    
    
    stages {       

         stage('Logging into AWS ECR') {
            steps {
                script {
                    LogToEcr()
                }
            }
        }   
    
         stage("Build and push image") {
            when { 
                changeset "src/*"
            }
            steps {
                script {
                    BuildPush(BRANCH_NAME, env.GIT_COMMIT, "nginx", BUILD_NUMBER)
                }
            }
        } 
    }
}