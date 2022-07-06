pipeline {
    agent any
    stages {
        stage('RunContainer') {
            steps {
                sh 'docker build . -t nginx'
                sh 'docker run --name nginx -d -p 9889:80 nginx'
            }
        }
        stage('Test-Status200') {
            steps {
                script {
                    sh '''
                        status_code=$(curl --write-out %{http_code} --silent --output /dev/null http://localhost:9889/index.html)
                        echo "Status code: $status_code"

                        if [[ $status_code -ge 200 ]]
                        then
                          curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='response: 200'
                        else
                          curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='something wron'
                        fi
                    '''
                }
            }
        }    
        stage('Test-MD5') {
            steps {
                script {
                    sh '''
                        online_md5="$(curl -sL http://localhost:9889/index.html | md5sum | cut -d ' ' -f 1)"
                        local_md5="$(md5sum "index.html" | cut -d ' ' -f 1)"

                        echo "Online MD5: $online_md5"
                        echo "Local MD5: $local_md5"

                        if [[ $online_md5 = $local_md5 ]]
                        then
                            echo "MD5 OK!"
                        else
                         exit 0
                        fi

                        if [[ $status_code -ge 200 ]]
                        then
                          curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='MD5 OK'
                        else
                          curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='MD5 not matched'
                        fi
                    '''
                }
            }
        }
        stage('Stage5-Docker-Cleare') {
            steps {
                sh '''
                    docker container stop nginx
                    docker container rm nginx
                    docker image rm nginx
                '''
            }
        }
}
    post {
        success {
                sh  '''
                curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC *Branch*: ${env.GIT_BRANCH} *Build* : OK *Published* = YES'
                '''
        }

        aborted {
                sh '''
                curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC *Branch*: ${env.GIT_BRANCH} *Build* : `Aborted` *Published* = `Aborted`'
                '''
        }
        failure {
            steps {
                sh '''
                    docker container stop nginx
                    docker container rm nginx
                    docker image rm nginx
                '''
                sh  '''
                curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC  *Branch*: ${env.GIT_BRANCH} *Build* : `not OK` *Published* = `no`'
                '''
            }
        }
    }
}
