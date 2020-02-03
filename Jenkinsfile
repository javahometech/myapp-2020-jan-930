pipeline{
    agent any
    stages{
        stage('Git'){
            steps{
                git credentialsId: 'javahometech', 
                    url: 'https://github.com/javahometech/myapp-2020-jan-930'
            }
        }
    }
}