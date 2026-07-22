pipeline {
    agent any

    environment {
        ACI_URL = "http://40.88.194.87"

        SERVERS = "ubuntu@44.214.89.39 ubuntu@3.82.187.145"

        STORAGE_ACCOUNT = "azblobpk"
        FILE_SHARE = "nginx-html"

        RESOURCE_GROUP = "rg-azuser7674_mml.local-hLEHv"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy Staging') {
            steps {
                withCredentials([
                    string(credentialsId: 'azure-storage-key',
                    variable: 'AZURE_STORAGE_KEY')
                ]) {

                    sh '''
                    az storage file upload \
                    --account-name ${STORAGE_ACCOUNT} \
                    --share-name ${FILE_SHARE} \
                    --source index.html \
                    --path index.html \
                    --account-key ${AZURE_STORAGE_KEY} \
                    --overwrite true
                    '''
                }
            }
        }

        stage('Test Staging') {
            steps {
                sh '''
                curl -f ${ACI_URL}

                curl ${ACI_URL} | grep "Version"
                '''
            }
        }

        stage('Deploy Production') {
            steps {

                sshagent(credentials: ['webservers-ssh-key']) {

                    sh '''

                    SSH_OPTS="-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"

                    for HOST in ${SERVERS}; do

                        rsync -az \
                        -e "ssh ${SSH_OPTS}" \
                        index.html \
                        ${HOST}:/tmp/

                        ssh ${SSH_OPTS} ${HOST} "
                        sudo cp /tmp/index.html /var/www/html/index.html &&
                        sudo systemctl reload apache2
                        "

                    done

                    '''
                }

            }
        }
    }
}