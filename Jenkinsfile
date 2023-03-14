// for slack purposes, define a variable called COLOR_MAP
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    
    environment {
        NEXUSPASS = credentials('nexuspass')
    }

    stages {
        
        // the purpose of this stage is taking the inputs from two parameters BUILD & TIME
        // it will be called in the following stage
        stage('Setup parameters') {
            steps {
                script { 
                    properties([
                        
                        parameters([
                            string(
                                defaultValue: '', 
                                name: 'BUILD', 
                            ),
							string(
                                defaultValue: '', 
                                name: 'TIME', 
                            )
                        ])
                    ])
                }
            }
		}

        stage('Ansible Deployment on production'){
            steps {
                // ansible plugin
                ansiblePlaybook([
                // inventory
                inventory   : 'ansible/prod.inventory',
                // playbook
                playbook    : 'ansible/site.yml',
                installation: 'ansible',
                colorized   : true,
                // credentials installed in Jenkins
			    credentialsId: 'appstagelogin',
                // if you didn't set it to true it will stop the stage
			    disableHostKeyChecking: true,   
                // extra variables for ansible playbook vars 
                extraVars   : [
                   	USER: "admin",
                    // store nexus password in jenkins and call it here
                    PASS: "$NEXUSPASS",
			        nexusip: "172.31.88.18",
			        reponame: "vprofile-hosted",
			        groupid: "QA",
                    // PLEASE BE ADVISED THE BELOW PARAMETERS ARE NOT EXISTED
                    // these WILL take it from the user
			        time: "${env.TIME}",
			        build: "${env.BUILD}",
                    // customized in line 113
                    artifactid: "vproapp",
			        vprofile_version: "vproapp-${env.BUILD}-${env.TIME}.war"
                ]
             ])
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