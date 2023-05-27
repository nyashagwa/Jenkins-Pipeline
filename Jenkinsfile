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
                //sh 'npm i -D cypress'
                sh 'npm install -D cypress@latest'
                echo "Executing integration tests using Cypress"
                echo "Cypress executing integration tests.... "
                //sh (script: 'NO_COLOR=1 /Users/jenipherg/NYASHA/node_modules/.bin/cypress run || true')
                sh "npx cypress run --key 6cc3e632-c6c1-4685-a0fa-79754b86df04 --spec "cypress/e2e/navigation.cy.js"
            }
            post{
            always{
                junit(testResults: 'cypress/results/results.xml', allowEmptyResults : true)
                archiveArtifacts(artifacts: 'cypress/videos/navigation.cy.js.mp4', fingerprint: true) 
            }
            success{
                emailext to: "ngwaradzimba@deakin.edu.au",
                subject: "Unit and Integration Tests Status Email",
                body: "Unit and Integration tests using Junit and Cypress respectively were successful ",
                attachLog: true
            }
            failure{
               emailext to: "ngwaradzimba@deakin.edu.au",
                subject: "Unit and Integration Tests Status Email",
                body: "Unit and Integration tests using Junit and Cypress respectively were unsuccessful ",
                attachLog: true
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
                snykSecurity(
                    snykInstallation: 'Synk-Security-Scanning',
                    snykTokenId: 'organisation-snyk-api-token')
            }
            post{
            success{
                emailext to: "ngwaradzimba@deakin.edu.au",
                subject: "Security Scan Status Email",
                body: "Security scan using Synk was successful",
                attachLog: true
            }
            failure{
                emailext to: "ngwaradzimba@deakin.edu.au",
                subject: "Security Scan Status Email",
                body: "Security scan using Synk was unsuccessful",
                attachLog: true
            }
        }
        }
        stage('Deploy to Staging') {
            steps{
                retry(2) {
                    echo "deploy the application to, $STAGING_ENVIRONMENT"
                    withAWS(region:'us-east-1',credentials:'AWS_Jenkins_Credentials')\
                    {
                    //sh 'aws s3 cp . s3://nyasha-staging-files/'
                    echo " Now uploading files for deployment to S3 bucket, please wait ....."
                    s3Upload(file: 'nyasha-deakin-unit-page.zip/.', bucket: 'nyasha-staging-files')
                    echo "Files uploaded to S3 bucket"
                    echo "Deploying to EC2 staging instance"
                    createDeployment(applicationName: 'nyasha-deakin-unit-page', deploymentGroupName: 'CodedeployNyasha',
                                     s3Bucket: 'nyasha-staging-files', waitForCompletion: true, ignoreApplicationStopFailures: true,
                                     s3BundleType: 'zip', s3Key: 'nyasha-deakin-unit-page.zip', fileExistsBehavior: 'OVERWRITE' )
                    echo "Deployment to Staging Complete"
                    }
                }
            }
        }
        stage('Integration Tests on Staging') {
            steps{
                echo "Run integration test on Staging using Cypress test automation tool "
                echo "Check Dependencies"
                sh 'npm ci'
                sh 'npm i -D cypress'
                echo "Cypress executing integration tests.... "
                sh (script: 'NO_COLOR=1 /Users/jenipherg/NYASHA/node_modules/.bin/cypress run || true')
                sh "npx cypress run --key 6cc3e632-c6c1-4685-a0fa-79754b86df04 --spec "cypress/e2e/navigation.cy.js"
            }
            post{
            always{
                junit(testResults: 'cypress/results/results.xml', allowEmptyResults : true)
                archiveArtifacts(artifacts: 'cypress/videos/navigation.cy.js.mp4', fingerprint: true) 
            }
            success{
                emailext to: "ngwaradzimba@deakin.edu.au",
                subject: "Integration Tests on Staging Status Email",
                body: "Integration tests on Staging using Cypress were successful ",
                attachLog: true
            }
            failure{
               emailext to: "ngwaradzimba@deakin.edu.au",
                subject: "Integration Tests on Staging Status Email",
                body: "Integration tests on Staging using Cypress were unsuccessful ",
                attachLog: true
            }
        }
        }
        stage('Deploy to Production') {
            steps{
                retry(2) {
                    echo "deploy the application to, $PRODUCTION_ENVIRONMENT"
                    //sh 'aws configure set region us-east-1'
                    withAWS(region:'us-east-1',credentials:'AWS_Jenkins_Credentials')\
                    {
                    echo " Now uploading files for deployment to S3 bucket, please wait ....."
                    s3Upload(file: 'nyasha-deakin-unit-page.zip/.', bucket: 'nyasha-prod-files')
                    echo "Files uploaded to S3 bucket"
                    echo "Deploying to EC2 Production instance"
                    createDeployment(applicationName: 'nyasha-deakin-unit-page', deploymentGroupName: 'CodedeployNyasha',
                                     s3Bucket: 'nyasha-prod-files', waitForCompletion: true, ignoreApplicationStopFailures: true,
                                     s3BundleType: 'zip', s3Key: 'nyasha-deakin-unit-page.zip', fileExistsBehavior: 'OVERWRITE' )
                    echo "Deployment to Production Complete"
                    }
                }
            }
        }

    }
}
