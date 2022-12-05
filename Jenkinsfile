pipeline{
    agent{
        node{
            label 'awsec2instance'
        }
    }
    stages{
        stage('Git checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/Chanti369/myrealtimeproject.git'
            }
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "adityakumarreddyvennapusa@gmail.com";  
		}
	}
}