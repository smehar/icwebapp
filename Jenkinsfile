pipeline{

    environment{
        IMAGE_NAME = "ic-webapp"
        IMAGE_TAG = "${BUILD_TAG}"
        USERNAME = "smehar"
        CONTAINER_NAME = "ic-webapp-test"
        STAGING_HOST = "34.207.185.66"
        //PROD_HOST =""
    }

    agent any

    stages{

        stage ('Build Image'){
            steps{
                script{
                    sh '''
                       docker build -t ${USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage ('Run a container and Test Image'){
            steps{
                script{
                    sh '''
                       docker stop ${CONTAINER_NAME} || true
                       docker rm ${CONTAINER_NAME} || true
                       docker run -d --name ${CONTAINER_NAME} -p 8085:8080 ${USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                       sleep 5
                       curl http://localhost:8085 | grep -iq "IC GROUP"
                    '''
                }
            }
        }

        stage ('save artifact and clean env'){
            environment{
                PASSWORD = credentials('dockerhub_password')
            }
            steps{
                script{
                    sh '''
                       docker login -u ${USERNAME} -p ${PASSWORD}
                       docker push ${USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                       docker stop ${CONTAINER_NAME}
                       docker rm ${CONTAINER_NAME}
                       docker rmi ${USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                       docker logout
                    '''
                }
            }
        }
        stage ('deploy app on Staging env'){
            agent any
            /*when {
                expression { GIT_BRANCH == 'origin/master'}
            }*/
            environment{
                HOST_IP = "${STAGING_HOST}"
                PGADMIN_PORT = "8082"
                ODOO_PORT = "8081"
                IC_PORT = "8080"
                NUSER = "ubuntu"
            }
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: "ec2_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    script{	
                        sh '''
                           #Install docker
                           curl -fsSL https://get.docker.com -o get-docker.sh
                           sh get-docker.sh
                           sudo usermod -aG docker ubuntu
                           #Install Docker compose
                           sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
                           sudo chmod +x /usr/local/bin/docker-compose
                           sleep 10 
                           scp -o StrictHostKeyChecking=no -i ${keyfile} $(pwd)/docker-compose.yml ${NUSER}@${HOST_IP}:/home/ubuntu/docker-compose.yml 
                           ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${HOST_IP} cd /home/ubuntu && docker-compose down || true
                           ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${HOST_IP} docker-compose up -d
                        '''
                    }
                }
            }
        }

       /* stage ('deploy app on Prod env'){
            agent any
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            environment{
                HOST_IP = "${STAGING_HOST}"
                PGADMIN_PORT = "8082"
                ODOO_PORT = "8081"
                IC_PORT = "8080"
            }
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: "ec2_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    script{
                        timeout(time: 15, unit: "MINUTES") {
                                input message: 'Do you want to approve the deploy in production?', ok: 'Yes'
                        }	
                        sh '''
                           scp -o StrictHostKeyChecking=no -i ${keyfile} $(pwd)/docker-compose.yml ${NUSER}@${HOST_IP}:/home/ubuntu/docker-compose.yml 
                           ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${HOST_IP} cd /home/ubuntu && docker-compose down || true
                           ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${HOST_IP} docker-compose up -d
                        '''
                    }
                }
            }
        }*/

    }

}
