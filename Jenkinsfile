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
		RELEASE_REPO = 'vprofile-release'
		CENTRAL_REPO = 'vpro-maven-central'
        // private ips for nexus EC2 server
		NEXUSIP = '172.31.29.38'
		NEXUSPORT = '8081'
        // nexus repo names
		NEXUS_GRP_REPO = 'vpro-maven-group'
        // nexus user on jenkins
        NEXUS_LOGIN = 'nexuslogin'
        // sonarqube configuration names on jenkins
        SONARSERVER = 'sonar-server'
        SONARSCANNER = 'sonarscanner'
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
    }
}