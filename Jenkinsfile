pipeline {
    agent any
    environment {
        DIRECTORY_PATH= '/Users/jenipherg/NYASHA PROGRAMS/Code'
        TESTING_ENVIRONMENT= 'NYASHA PRE-PROD TEST'
        PRODUCTION_ENVIRONMENT= 'Nyasha Gwaradzimba'
    }
    stages {
        stage('Build') {
            steps{
                echo "fetch the source code from, $DIRECTORY_PATH"
                echo "compile the code and generate any necessary artifacts"
            }
            post{
            success{
                mail to: "ngwaradzimba@deakin.edu.au",
                subject: "Build Status Email",
                body: "Build was successful"
            }
            failure{
                echo "pipleline execution failed"
            }
        }
        }
        stage('Test') {
            steps{
                echo "unit tests"
                echo "integration tests"
            }
        }
        stage('Code Quality Check') {
            steps{
                echo "check the quality of the code"
            }      
        }
        stage('Deploy') {
            steps{
                retry(2) {
                    echo "deploy the application to, $TESTING_ENVIRONMENT"
                }
                //timeout(time: 3, unit: 'SECONDS') {
                //    sleep 5
                //}
                
            }
        }
        stage('Approval') {
            steps{
                sleep 10
            }
        }
        stage('Deploy to Production') {
            steps{
                echo "deploy code to, PRODUCTION_ENVIRONMENT "
            }
        }
        
    }
    post{
            always{
                echo "======always======="
            }
            success{
                echo "pipeline executed successfully"
            }
            failure{
                echo "pipleline execution failed"
            }
        }
}