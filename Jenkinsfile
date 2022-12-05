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
                git branch: 'main', url: 'https://github.com/Chanti369/myrealtimeproject.git'
            }
        }
        stage('unit testing'){
            steps{
                script{
                    sh 'mvn clean test'
                }
            }
        }
        stage('integartion testing'){
            steps{
                script{
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
        stage('mvn install'){
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
        stage('Quality gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube'
                }
            }
        }
        stage('nexus artifact uploader'){
            steps{
                script{
                    def readpom = readMavenPom file: 'pom.xml'
                    def readversion = readpom.version
                    def readrepo = readversion.endsWith("SNAPSHOT") ? "myprojectrepo-snapshot" : "myprojectrepo-release"
                    nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/Uber.jar', type: 'jar']], credentialsId: 'nexus', groupId: 'com.javatechie', nexusUrl: '13.127.1.5:8081', nexusVersion: 'nexus3', protocol: 'http', repository: readrepo, version: readversion
                }
            }
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "adityakumarreddyvennapusa@gmail.com";  
		}
	}
}