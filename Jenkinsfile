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
                def pomFile = readMavenPom file: 'pom.xml'
                
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
                    repository: 'myapp-release', 
                    version: "${pomFile.version}"

            }
        }
    }
}