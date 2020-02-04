pipeline{
    agent any
    tools {
        maven 'maven3'
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
                sshagent(['tomcat-dev']) {
                    sh "scp -o StrictHostKeyChecking=no target/myweb*.war ec2-user@172.31.0.38:/opt/tomcat8/webapps/myweb.war"
                    sh "ssh ec2-user@172.31.0.38 /opt/tomcat8/bin/shutdown.sh"
                    sh "ssh ec2-user@172.31.0.38 /opt/tomcat8/bin/startup.sh"
                }
            }
        }
    }
}