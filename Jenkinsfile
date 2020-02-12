pipeline{

    agent any
    tools {
        maven 'maven3'
    }
    parameters {
        choice choices: ['dev', 'uat', 'prod'], description: 'Choose the environment to deploy your artifact', name: 'appEnv'
        booleanParam defaultValue: false, description: 'Do you wanna rollback the deployment?', name: 'rollback'
    }
    stages{

        stage('Build And Sonar'){
            parallel{
                stage('Maven Build'){
                    when{
                        expression{
                            params.rollback  == false
                            params.appEnv == 'dev'
                        }
                    }
                    steps{
                        sh script: 'mvn clean package'
                    }
                }

                stage('Sonar Publish'){
                    steps{
                        withSonarQubeEnv('sonar7') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                }

            }

        }
        
        

        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage('Upload Artifacts to Nexus'){
            when{
                expression{
                    params.rollback == false
                    params.appEnv == 'dev'
                }
            }
            steps{
                script{
                    def pomFile = readMavenPom file: 'pom.xml'
                    def nexusRepo = "myapp-release"
                    if(pomFile.version.endsWith("SNAPSHOT")){
                        nexusRepo = "myapp-snapshot"
                    }
                    nexusArtifactUploader artifacts: [
                            [
                                artifactId: 'myweb', 
                                classifier: '', file: 
                                "target/myweb-${pomFile.version}.war", type: 'war'
                            ]
                        ], 
                        credentialsId: 'nexus3', 
                        groupId: 'in.javahome', 
                        nexusUrl: '172.31.34.232:8081', 
                        nexusVersion: 'nexus3', 
                        protocol: 'http', 
                        repository: "${nexusRepo}", 
                        version: "${pomFile.version}"

                }

            }
        }

        stage("Deploy - Dev"){
            when{
                expression{
                    params.appEnv == 'dev'
                }
            }
            steps{
                script{
                    sshagent(['tomcat-dev']) {
                    def warFilePath = "target/myweb*.war"
                    if(params.rollback == true){
                        echo "Rollbacking to ${env.JENKINS_HOME}/jobs/${env.JOB_NAME}/builds/${currentBuild.previousSuccessfulBuild.number-1}/archive/${warFilePath}"
                        warFilePath = "${env.JENKINS_HOME}/jobs/${env.JOB_NAME}/builds/${currentBuild.previousSuccessfulBuild.number-1}/archive/${warFilePath}"
                    }
                    sh "scp -o StrictHostKeyChecking=no ${warFilePath} ec2-user@172.31.0.38:/opt/tomcat8/webapps/myweb.war"
                    sh "ssh ec2-user@172.31.0.38 /opt/tomcat8/bin/shutdown.sh"
                    sh "ssh ec2-user@172.31.0.38 /opt/tomcat8/bin/startup.sh"
                }
                }
            }
        }

        stage("Deploy - UAT"){
            when{
                expression{
                    params.appEnv == 'uat'
                }
            }
            steps{
                echo "Deploy to UAT"
            }
        }

        stage("Deploy - Prod"){
            when{
                expression{
                    params.appEnv == 'prod'
                }
            }
            steps{
                input message: 'Do you wanna deploy to Prod?', submitter: 'prakash'
                echo "Deploy to Prod"
            }
        }
    }
    post{
        success{
            
            script{
                if(params.rollback == false){
                    def pomFile = readMavenPom file: 'pom.xml'
                    archiveArtifacts "target/myweb-${pomFile.version}.war"
                }
            }
        }
        failure {
            mail  body: 'The following Jenkins Job failed', subject: 'Jenkins Job', to: 'javahometechnologies@gmail.com'
        }
    }
}