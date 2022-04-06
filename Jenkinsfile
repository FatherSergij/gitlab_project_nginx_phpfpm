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
                    BuildPush(BRANCH_NAME, env.GIT_COMMIT, "nginx")
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
                build job: 'Job_deploy', parameters: [string(name: 'BranchRun_dev', value: env.BRANCH_NAME), 
                  string(name: 'ImageTag_dev', value: GIT_COMMIT),
                  string(name: 'ServiceRun_dev', value: "nginx")]
            }
        }
    }
}