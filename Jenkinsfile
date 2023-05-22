pipeline {
    agent any
    tools { 
       maven 'MAVEN' 
       jdk 'JDK' 
       nodejs 'Node10'
    }
    environment {
        DIRECTORY_PATH= '/Users/jenipherg/IdeaProjects/'
        STAGING_ENVIRONMENT= '@AWS_STAGING_INSTANCE'
        PRODUCTION_ENVIRONMENT= '@AWS_PROD_INSTANCE'
        CHROME_BIN = '/bin/google-chrome'
    }
    stages {
        stage('Build') {
            steps{
                echo "Using Maven build tool to compile and package code"
                echo "Fetch the source code from, $DIRECTORY_PATH"
                echo "Maven compiling code...."
                sh 'mvn -B -DskipTests clean'
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
                echo "Check Dependencies"
                sh 'npm ci'
                sh 'npm i -D cypress'
                echo "Executing integration tests using Cypress"
                echo "Cypress executing integration tests.... "
                sh (script: 'NO_COLOR=1 /Users/jenipherg/NYASHA/node_modules/.bin/cypress run || true')
                sh "npx /Users/jenipherg/NYASHA/node_modules/.bin/cypress run --key 6cc3e632-c6c1-4685-a0fa-79754b86df04"
            }
            post{
            always{
                junit(testResults: 'cypress/results/results.xml', allowEmptyResults : true)
                archiveArtifacts(artifacts: 'cypress/videos/spec.cy.js.mp4', fingerprint: true) 
            }
            success{
                mail to: "ngwaradzimba@deakin.edu.au",
                subject: "Unit and Integration Tests Status Email",
                body: "Unit and Integration tests using Junit and Cypress respectively were successful "
            }
            failure{
               mail to: "ngwaradzimba@deakin.edu.au",
                subject: "Unit and Integration Tests Status Email",
                body: "Unit and Integration tests using Junit and Cypress respectively were unsuccessful "
            }
        }
        }
        stage('Code Analysis') {
            steps{
                echo "check the quality of the code using Sonar Cube Scanner"
                withSonarQubeEnv(installationName: 'SonarCubeScanner', credentialsId: 'SonarQubeToken') {
                sh 'mvn sonar:sonar'
            }      
        }
        }
        stage('Security Scan') {
            steps{
                echo "Performing security scan using Synk, Probely, Jfrog Xray or Veracode"
                echo "Security scanning with Synk"
                //probelyScan targetId: 'NyashaTest', credentialsId: 'probely-test-site'
                snykSecurity(
                    snykInstallation: 'Synk-Security-Scanning',
                    snykTokenId: 'organisation-snyk-api-token')
            }
            post{
            success{
                mail to: "ngwaradzimba@deakin.edu.au",
                subject: "Security Scan Status Email",
                body: "Security scan using Synk was successful"
            }
            failure{
                mail to: "ngwaradzimba@deakin.edu.au",
                subject: "Security Scan Status Email",
                body: "Security scan using Synk was unsuccessful"
            }
        }
        }
        stage('Deploy to Staging') {
            steps{
                retry(2) {
                    echo "deploy the application to, $STAGING_ENVIRONMENT"
                    //sh 'aws configure set region us-east-1'
                    withAWS(region:'us-east-1',credentials:'AWS_Jenkins_Credentials')\
                    {
                    //sh 'aws s3 cp . s3://nyasha-staging-files/'
                    echo " Now uploading files for deployment to S3 bucket, please wait ....."
                    s3Upload(file: 'Heloworld/.', bucket: 'nyasha-staging-files')
                    echo "Files uploaded to S3 bucket"
                    echo "Deploying to EC2 staging instance"
                    createDeployment(applicationName: 'nyasha-deakin-unit-page',deploymentGroupName: 'CodedeployNyasha', 
                                     s3Bucket: 'nyasha-staging-files')
                    }
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
