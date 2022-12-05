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
                    nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'nexus', groupId: 'com.javatechie', nexusUrl: '13.127.1.5:8081', nexusVersion: 'nexus3', protocol: 'http', repository: readrepo, version: readversion
                }
            }
        }
        stage('docker build image'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                        sh 'docker login -u $dockerusername -p $dockerpassword'
                        sh 'docker build -t $JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername/$JOB_NAME:latest'
                    }
                }
            }
        }
        stage('docker push'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                        sh 'docker login -u $dockerusername -p $dockerpassword'
                        sh 'docker push $dockerusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker push $dockerusername/$JOB_NAME:latest'
                    }
                }
            }
        }
        stage('indentifying misconfigs using datree'){
            steps{
                script{
                    dir('kubernetes/myapp'){
                        withEnv(['DATREE_TOKEN=22ecd219-bce0-4cb8-8a9a-efab1589ab1d']) {
                            sh 'datree test *.yaml --only-k8s-files'
                        }
                    }
                }
            }
        }
        stage('helm chart to nexus repo'){
            steps{
                script{
                    dir('kubernetes/'){
                        withCredentials([usernamePassword(credentialsId: 'nexushelm', passwordVariable: 'nexuspassword', usernameVariable: 'nexususername')]) {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d ":" -f 2 | tr -d " ")
                            tar -czvf myapp-${helmversion}.tgz /myapp
                            curl -u $nexususername:$nexuspassword http://13.127.1.5:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }
                    }
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