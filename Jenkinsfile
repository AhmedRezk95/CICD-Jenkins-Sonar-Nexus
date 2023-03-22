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
        SNAP_REPO = 'vprofile-snapshot'
        // username / password for nexus
		NEXUS_USER = 'admin'
		NEXUS_PASS = 'admin'
        // nexus repo names
		RELEASE_REPO = 'vprofile-hosted'
		CENTRAL_REPO = 'vprofile-proxy'
        // private ips for nexus EC2 server
		NEXUSIP = '172.31.88.18'
		NEXUSPORT = '8081'
        // nexus repo names
		NEXUS_GRP_REPO = 'vprofile-group'
        // nexus user on jenkins
        NEXUS_LOGIN = 'nexuslogin'
        // sonarqube configuration names on jenkins
        SONARSERVER = 'sonar-server'
        SONARSCANNER = 'sonarscanner'
        // Set Artifact Name
        ARTIFACT_NAME = "vprofile-v${BUILD_ID}.war"
        // Set S3 Bucket
        AWS_S3_BUCKET = 'cicd-jenkins-beanstalk'
        // BEANSTALK APP NAME
        AWS_EB_APP_NAME = 'jenkins-beanstalk-app'
        // BEANSTALK APP ENV
        AWS_EB_ENVIRONMENT = 'Jenkinsbeanstalkapp-env'
        // BEANSTALK APP VERSIONS
        AWS_EB_APP_VERSION = "${BUILD_ID}"
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving."
                    // use archiveArtifacts plugin to artifact your build, search for all .war files
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test'){
            steps {
                // just test
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis'){
            steps {
                // check tool for your build
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }

            steps {
               // check LECTURE #54
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                   sh 'sleep 30'
              }
            }
        }

        stage("Quality Gate") {
            // using sonarqube quality gate
            // QG: timeout after one hour
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("UploadArtifact"){
            steps{
                // use nexus plugin in jenkins
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                  // groupId is just the Header directory that will be created in Nexus repo
                  groupId: 'QA',
                  // artiact pattern will be versioned based on jenkins (job build number-timestamp plugin)
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${RELEASE_REPO}",
                  credentialsId: "${NEXUS_LOGIN}",
                  // some info about the artifact that we want to upload
                  // artifactId act as a prefix for the artifact 
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }

        stage('Deploy to Stage Bean'){
          steps {
            withAWS(credentials: 'aws', region: 'us-east-1') {
               // copy artifact from Jenkins server to S3
               sh 'aws s3 cp ./target/vprofile-v2.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
               // Create beanstalk version from Artifact
               sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
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