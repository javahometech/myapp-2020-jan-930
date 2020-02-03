pipeline{
    agent any
    stages{
        stage('Maven Build'){
            steps{
                sh script: 'mvn clean package'
            }
        }
    }
}