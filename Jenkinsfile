pipeline {
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
                    if (branch == 'develop') {
                        BuildPush(BRANCH_NAME, env.GIT_COMMIT, "nginx", BUILD_NUMBER)
                    } else {
                        BuildPush(BRANCH_NAME, "latest", "nginx", BUILD_NUMBER)
                    }
                }
            }
        } 


        stage("Deploy on k8s") {
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
                build job: 'Job_deploy', parameters: [string(name: 'BranchRun_dev', value: env.BRANCH_NAME), 
                  string(name: 'ImageTag_dev', value: GIT_COMMIT),
                  string(name: 'ServiceRun_dev', value: "nginx")]
            }
        }
    }
}