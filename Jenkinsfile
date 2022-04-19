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
                    if (BRANCH_NAME == "develop") {
                        BuildPush(BRANCH_NAME, "latest", "node", BUILD_NUMBER)
                    } else {
                        BuildPush(BRANCH_NAME, env.GIT_COMMIT, "node", BUILD_NUMBER) //jj
                    }
                }
            }
        } 


        stage("Deploy on k8s") {
            when { 
                branch 'develop'
                changeset "src/*"
            } 
            steps {
                build job: 'Job_deploy', parameters: [string(name: 'BranchRun_dev', value: env.BRANCH_NAME), 
                  string(name: 'ImageTag_dev', value: "latest")]
            }
        }        
    }
}