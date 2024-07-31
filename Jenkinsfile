pipeline{
    agent any
    environment {
        git_repo = "https://github.com/rithery/nestjs-mongodb-basic"
        docker_hub_password = credentials('docker_hub')
    }
    parameters {
        choice(name: 'APP_ENV', choices: ['uat','preprod','prod'], description: 'Please choose enviroment to build')
    }
    stages{
        stage("Configure"){
            steps{
                sh """
                    git clone ${git_repo}
                """
            }
        }
        stage("Build"){
            steps{
                sh """
                    cd nestjs-mongodb-basic
                    docker build . -t rithery/nestjs-mongo-api:${APP_ENV}-${BUILD_NUMBER}
                    docker login -u rithery -p ${docker_hub_password}
                    docker push rithery/nestjs-mongo-api:${APP_ENV}-${BUILD_NUMBER}
                """
            }
        }
    }
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