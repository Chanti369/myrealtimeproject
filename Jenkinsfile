pipeline{
    agent{
        node{
            label 'awsec2instance'
        }
    }
    tools{
        maven 'MAVEN'
    }
    stages{
        stage('Git checkout'){
            steps{
                script{
                    git branch: 'main', url: 'https://github.com/Chanti369/myrealtimeproject.git'
                }
            }
        }
        stage('mvn clean install'){
            steps{
                script{
                    sh 'mvn clean install'
                }
            }
        }
        stage('static code analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube') {
                        sh 'mvn clean install sonar:sonar'
                    }
                }
            }
        }
    }
}