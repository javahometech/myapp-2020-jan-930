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
                nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'myweb', 
                            classifier: '', file: 
                            'target/myweb-8.5.0.war', type: 'jar'
                        ]
                    ], 
                    credentialsId: 'nexus3', 
                    groupId: 'in.javahome', 
                    nexusUrl: '172.31.34.232:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'myapp-release', 
                    version: '8.5.0'

            }
        }
    }
}