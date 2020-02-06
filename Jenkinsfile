pipeline{

    agent any
    tools {
        maven 'maven3'
    }
    parameters {
        booleanParam defaultValue: false, description: 'Do you wanna rollback the deployment?', name: 'rollback'
    }
    stages{
        stage('Maven Build'){
            steps{
                sh script: 'mvn clean package'
            }
        }
        stage('Upload Artifacts to Nexus'){
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
    }
    post{
        success{
            script{
                def pomFile = readMavenPom file: 'pom.xml'
                archiveArtifacts "target/myweb-${pomFile.version}.war"
            }
        }
        failure {
            mail  body: 'The following Jenkins Job failed', subject: 'Jenkins Job', to: 'javahometechnologies@gmail.com'
        }
    }
}