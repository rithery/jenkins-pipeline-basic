pipeline{
    agent any
    environment {
        git_repo = "https://github.com/rithery/nestjs-mongodb-basic"
        digitalocean_registry = "registry.digitalocean.com/registry-devops/nestjs-mongodb-basic"
        digitalocean_token = credentials('digitalocean_token')
        docker_hub_password = credentials('docker_hub')
        telegram_token = credentials('Telegram_Token')
        telegram_chatID = credentials('Telegram_ChatID')
        cloudflare_email = 'rithery11@gmail.com'
        cloudflare_api_key = credentials('cloudflare_api_key')
        SERVER_IP = '165.22.241.82'
        DOMAIN = 'api.rithe.cloud'
        ROOT_DOMAIN = 'rithe.cloud'
        PROJECT_NAME = 'NestJS Mongo'
        SERVICE_NAME = 'API'
        email = 'ab123@gmail.com'
    }
    parameters {
        choice(name: 'APP_ENV', choices: ['uat','preprod','prod'], description: 'Please choose enviroment to build')
        text(name: 'Release_Note', defaultValue: 'Deploy application', description: 'Need your note before click build')
    }
    stages{
        stage("Configure"){
            steps{
                sh """
                    rm -rf nestjs-mongodb-basic
                    git clone ${git_repo}
                """
            }
        }
        stage("Build"){
            steps{
                sh """
                    cd nestjs-mongodb-basic
                    docker build . -t ${digitalocean_registry}:${APP_ENV}-${BUILD_NUMBER}
                    docker login registry.digitalocean.com -u ${digitalocean_token} -p ${digitalocean_token}
                    docker push ${digitalocean_registry}:${APP_ENV}-${BUILD_NUMBER}
                """
            }
        }
        stage("Deploy"){
            steps{
                sh """
                    ssh root@165.22.241.82 "cd /srv;\
                                            docker login registry.digitalocean.com -u ${digitalocean_token} -p ${digitalocean_token};\
                                            sed -i 's/1/${APP_ENV}-${BUILD_NUMBER}/g' .env;\
                                            docker-compose up -d --build;\
                                            sed -i 's/${APP_ENV}-${BUILD_NUMBER}/1/g' .env"
                """
            }
        }
        stage("Install Certbot and Setup SSL") {
            steps {
                script {
                    sh """
                        ssh root@${SERVER_IP} "apt install -y certbot python3-certbot-nginx && \
                                              certbot --nginx -d ${DOMAIN} --email ${email} --agree-tos --non-interactive && \
                                              systemctl reload nginx"
                    """
                }
            }
        }
        stage("Configure Cloudflare") {
            steps {
                script {
                    sh """
                        curl -X POST "https://api.cloudflare.com/client/v4/zones" \
                            -H "X-Auth-Email: ${cloudflare_email}" \
                            -H "X-Auth-Key: ${cloudflare_api_key}" \
                            -H "Content-Type: application/json" \
                            --data '{"name":"${ROOT_DOMAIN}"}'

                        ZONE_ID=\$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones?name=${ROOT_DOMAIN}" \
                            -H "X-Auth-Email: ${cloudflare_email}" \
                            -H "X-Auth-Key: ${cloudflare_api_key}" \
                            -H "Content-Type: application/json" | jq -r '.result[0].id')

                        curl -X POST "https://api.cloudflare.com/client/v4/zones/\${ZONE_ID}/dns_records" \
                            -H "X-Auth-Email: ${cloudflare_email}" \
                            -H "X-Auth-Key: ${cloudflare_api_key}" \
                            -H "Content-Type: application/json" \
                            --data '{"type":"A","name":"${DOMAIN}","content":"${SERVER_IP}","ttl":120,"proxied":true}'
                    """
                }
            }
        }
        stage("Reload Nginx") {
            steps {
                script {
                    sh """
                        ssh root@${SERVER_IP} "systemctl reload nginx"
                    """
                }
            }
        }
    }
    post{
        always{
            script {
                sh """
                    curl -s -X POST https://api.telegram.org/bot${telegram_token}/sendMessage\
                            -d chat_id=${telegram_chatID} \
                            -d parse_mode="HTML" \
                            -d text="<b>Stage</b>: Deploy ${PROJECT_NAME} on service ${SERVICE_NAME} \
                            %0A<b>Status</b>: ${currentBuild.currentResult} \
                            %0A<b>Version</b>: ${APP_ENV}-${BUILD_NUMBER} \
                            %0A<b>Environment</b>: ${APP_ENV} \
                            %0A<b>DOMAIN</b>: ${DOMAIN}
                            %0A<b>User Build</b>: ${BUILD_USER} \
                            %0A<b>Release Note</b>: ${Release_Note} "
                """
            }
        }
    }
}
