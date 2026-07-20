pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "fabian1243"  // replace this with your docker-id
        DOCKER_IMAGE = "liora_devops_exam"
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
    }
    agent any // Jenkins will be able to select all available agents
    stages {
        stage('Docker compose up'){ // docker build image stage
            steps {
                script {
                    sh '''
                        docker compose up -d --build
                        sleep 6
                    '''
                }
            }
        }

        stage('Test Acceptance'){ // lanzamos el comando curl para validar que el contenedor responde a la solicitud
            steps {
                script {
                    sh 'curl localhost:8080'
                }
            }
        }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker compose push
                    '''
                }
            }
        }

        stage('Deployment in dev'){  
            environment {  
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins  
            }  
            steps {  
                script {  
                    sh '''  
                        rm -Rf .kube  
                        mkdir .kube  
                        cat $KUBECONFIG > .kube/config  
                        cp charts/values.yaml values.yml  
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml  
                        helm upgrade --install app fastapi --values=values.yml --namespace dev  
                    '''  
                }  
            }  
        }

        stage('Deployment in staging'){  
            environment {  
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins  
            }  
            steps {  
                script {  
                    sh '''  
                        rm -Rf .kube  
                        mkdir .kube  
                        cat $KUBECONFIG > .kube/config  
                        cp charts/values.yaml values.yml  
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml  
                        helm upgrade --install app fastapi --values=values.yml --namespace staging  
                    '''  
                }  
            }  
        }

        stage('Deployment in prod'){  
            environment {  
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins  
            }  
            steps {  
                // Create an Approval Button with a timeout of 15minutes.  
                // this require a manual validation in order to deploy on production environment  
                timeout(time: 15, unit: "MINUTES") {  
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'  
                }  
                script {  
                    sh '''  
                        rm -Rf .kube  
                        mkdir .kube  
                        cat $KUBECONFIG > .kube/config  
                        cp charts/values.yaml values.yml  
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml  
                        helm upgrade --install app fastapi --values=values.yml --namespace prod  
                    '''  
                }  
            }  
        }  
    }  
    post { // send email when the job has failed  
        failure {  
            echo "This will run if the job failed"  
            mail to: "ab95cd@yahoo.de",  
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",  
                body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"  
        }  
    }  
}  