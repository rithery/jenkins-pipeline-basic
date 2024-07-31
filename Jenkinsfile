pipeline{
    agent any
    environment {
        git_repo = "https://github.com/rithery/nestjs-mongodb-basic"
        digitalocean_registry = "registry.digitalocean.com/registry-devops/nestjs-mongodb-basic"
        digitalocean_token = credentials('digitalocean_token')
        docker_hub_password = credentials('docker_hub')
        telegram_token = credentials('Telegram_Token')
        telegram_chatID = credentials('Telegram_ChatID')
        PROJECT_NAME = 'NestJS Mongo'
        SERVICE_NAME = 'NestJS Mongo API'
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
                            %0A<b>Application URL</b>: https://jenkins.rithe.cloud \
                            %0A<b>User Build</b>: ${BUILD_USER} \
                            %0A<b>Release Note</b>: ${Release_Note} "
                """
            }
        }
    }
}
