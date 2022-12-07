pipeline{
    agent{
        node{
            label 'awsec2instance'
        }
    }
    stages{
        stage('Git checkout'){
            steps{
                script{
                    git branch: 'main', url: 'https://github.com/Chanti369/myrealtimeproject.git'
                }
            }
        }
    }
}