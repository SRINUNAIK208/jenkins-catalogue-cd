pipeline {
    agent {
        label 'AGENT-1'
    }
    options{
        timeout(time:30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = '388343452532'
        project = 'roboshop'
        component = 'catalogue'
        //SCANNER_HOME = tool 'Sonar-scanner'
    }  
    parameters {
        string(name: 'appversion', description: 'application image')
        choice(name: 'deploy_to', choices: ['dev', 'qa','prod'], description: 'pick the environment')
    }
    stages{
        stage('Deploy'){
            steps{
                withAWS(credentials: 'aws-cred', region: 'us-east-1'){
                    sh """
                      aws eks update-kubeconfig --region ${REGION} --name ${project}
                      kubectl get nodes 
                      sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                      helm upgrade --install ${component} -f values-${params.deploy_to}.yaml -n ${project} .
                    """
                }
            }
        }
        stage('check the deployment status'){
            steps{
                script{
                    withAWS(credentials: 'aws-cred', region: 'us-east-1'){
                        def DeploymentStatus = sh(returnStdout: true, script: 'kubectl rollout status deployment/catalogue --timeout=30s -n $project || echo failed').trim()
                        if (DeploymentStatus.contains('successfully rolled out'))
                        {

                            echo "Deployment is success"
                        }
                        else{
                            sh """
                            helm rollback ${component} -n ${project}
                            sleep 20 
                            """
                            def rollbackStatus = sh(returnStdout: true, script: 'kubectl rollout status deployment/catalogue --timeout=30s -n $project || echo failed').trim()
                            if (rollbackStatus.contains('successfully rolled out'))
                            {
                                echo "Deployment is failed, rollback is success"
                            }
                            else {
                                error "Deployment is failed and rollback also failed, its emergency"
                            }

                        }
                    }
                }
            }
        }

    }
    post {
        always {
            echo "hello i am always block"
        }
        success {
            echo "i am success"
        }
        failure {
            echo "i am failed"
        }
    }
}