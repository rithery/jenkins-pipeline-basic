pipeline{
    agent any
    environment {
        git_repo = "https://github.com/rithery/jenkins-pipeline-basic"
    }
    parameters {
        choice(name: 'APP Enviroment', choices: ['uat','preprod','prod'], description: 'Please choose enviroment to build')
    }
    stages{
        stage("Configure"){
            steps{
                echo "Configure Stage"
            }
        }
        stage("Deploy Production"){
            steps{
                echo "Production Stage"
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