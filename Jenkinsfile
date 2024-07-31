pipeline{
    agent any
    environment {
        git_repo = "https://github.com/rithery/nestjs-mongodb-basic"
        digitalocean_registry = "registry.digitalocean.com/registry-devops/nestjs-mongodb-basic"
        digitalocean_token = credentials('digitalocean_token')
        docker_hub_password = credentials('docker_hub')
    }
    parameters {
        choice(name: 'APP_ENV', choices: ['uat','preprod','prod'], description: 'Please choose enviroment to build')
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
        stage("Deploy UAT"){
            steps{
                sh """
                    ssh root@165.22.241.82 "cd /srv;\
                                            docker login registry.digitalocean.com -u ${digitalocean_token} -p ${digitalocean_token} \
                                            sed -i 's/1/${APP_ENV}-${BUILD_NUMBER}/g' .env;\
                                            docker compose up-d --build;\
                                            sed -i 's/${APP_ENV}-${BUILD_NUMBER}/1/g' .env"
                """
            }
        }
    }
                    // docker login -u rithery -p ${docker_hub_password}
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}
