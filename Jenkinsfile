pipeline {
    agent any
    tools { 
       maven 'MAVEN' 
       jdk 'JDK' 
    }
    environment {
        DIRECTORY_PATH= '/Users/jenipherg/IdeaProjects/'
        STAGING_ENVIRONMENT= '@AWS_STAGING_INSTANCE'
        PRODUCTION_ENVIRONMENT= '@AWS_PROD_INSTANCE'
    }
    stages {
        stage('Build') {
            steps{
                echo "Using Maven build tool to compile and package code"
                echo "Fetch the source code from, $DIRECTORY_PATH"
                echo "Maven compiling code...."
                sh 'mvn -B -DskipTests clean package'
                echo "Maven packaging code...."
            }
            post {
                always {
                   junit(
                     allowEmptyResults: true,
                     testResults: '*/test-reports/.xml'
                 )
               }
            } 
        }
        stage('Unit and Integration Tests') {
            steps{
                echo "Run unit tests using JUnit test automation tool "
                echo "JUnit executing unit tests...."
                echo "Executing integration tests using Katalon"
                echo "Katalon executing integration tests.... "
            }
            post{
            success{
                mail to: "ngwaradzimba@deakin.edu.au",
                subject: "Unit and Integration Tests Status Email",
                body: "Unit and Integration tests using Junit and Katalon respectively were successful "
            }
            failure{
                echo "Unit and Integration Tests failed"
            }
        }
        }
        stage('Code Analysis') {
            steps{
                echo "check the quality of the code"
                withSonarQubeEnv(installationName: 'SonarCubeScanner', credentialsId: 'SonarQubeToken') {
                sh 'mvn clean package sonar:sonar'
            }      
        }
        }
        stage('Security Scan') {
            steps{
                echo "Performing security scan"
            }
            post{
            success{
                mail to: "ngwaradzimba@deakin.edu.au",
                subject: "Security Scan Status Email",
                body: "Security scan was successful"
            }
            failure{
                echo "Security scan failed"
            }
        }
        }
        stage('Deploy to Staging') {
            steps{
                retry(2) {
                    echo "deploy the application to, $STAGING_ENVIRONMENT"
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
}
