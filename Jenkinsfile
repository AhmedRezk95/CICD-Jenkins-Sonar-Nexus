// based on previous work on staging pipeline, the main goal is to deploy the last successful artifact from staging pipeline

// (MUST): we need to add the last successful build job from jenkins called "cicd-jenkins-bean-stg"
// define a variable to represent the job number
def buildNumber = Jenkins.instance.getItem('cicd-jenkins-bean-stg').lastSuccessfulBuild.number

// for slack purposes, define a variable called COLOR_MAP
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "maven3"
        jdk "jdk8"
    }
    
    environment {
        // Set S3 Bucket
        AWS_S3_BUCKET = 'cicd-jenkins-beanstalk'
        // BEANSTALK APP NAME
        AWS_EB_APP_NAME = 'jenkins-beanstalk-app'
        // BEANSTALK APP ENV
        AWS_EB_ENVIRONMENT = 'Jenkinsbeanstalkapp-env'
        // BEANSTALK APP VERSIONS
        AWS_EB_APP_VERSION = "${buildNumber}"
    }

    stages {
        stage('Deploy to Stage Bean'){
          steps {
            withAWS(credentials: 'aws', region: 'us-east-1') {
               // we only need to deploy the last successful build version from Staging pipeline to the production
               // update beanstalk environment with the new version
               sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
          }
        }
    }
    // add slack for notification
    post {
        always {
            echo 'Slack Notifications.'
            // set your channel name
            slackSend channel: '#cicd',
                /*
                    For more information about Jenkins global variable "currentBuild" please visit the following:
                    https://kb.novaordis.com/index.php/Jenkins_currentBuild

                    COLOR_MAP Variable is defined previously in line 2
                */
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}