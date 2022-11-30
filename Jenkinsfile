pipeline{
    agent any
    tools{
        maven "MAVEN"
    }
    stages{
        stage('Git Checkout'){
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
                    withSonarQubeEnv(credentialsId: 'sonarqube'){
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
        stage('send artifact to nexus'){
            steps{
                script{
                    def readpom = readMavenPom file: 'pom.xml'
                    def readversion = readpom.version
                    def readnexusrepo = readversion.endsWith("SNAPSHOT") ? "myrealtimeproject-snapshot" : "myrealtimeproject-release"
                    nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'nexus_cred', groupId: 'com.javatechie', nexusUrl: '13.126.96.40:8081', nexusVersion: 'nexus3', protocol: 'http', repository: readnexusrepo, version: readversion
                }
            }
        }
        stage('docker build image'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_cred', variable: 'docker_cred')]){
                        sh 'docker login -u 7995323158 -p ${docker_cred}'
                        sh 'docker build -t $JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID 7995323158/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID 7995323158/$JOB_NAME:latest'
                    }
                }
            }
        }
        stage('push docker images'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_cred', variable: 'docker_cred')]){
                        sh 'docker login -u 7995323158 -p ${docker_cred}'
                        sh 'docker push 7995323158/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker push 7995323158/$JOB_NAME:latest'

                    }
                }
            }
        }
        stage('indentifying misconfigs using datree'){
            steps{
                script{
                    dir('kubernetes/myapp/'){
                        withEnv(['DATREE_TOKEN=22ecd219-bce0-4cb8-8a9a-efab1589ab1d']) {
                            sh 'datree test *.yaml --only-k8s-files'
                        }
                    }    

                }
            }

        }
        stage('push chart to nexus repo'){
            steps{
                script{
                   withCredentials([usernamePassword(credentialsId: 'nexus1', passwordVariable: 'usernamepasswd', usernameVariable: 'nexususername')]){
                     dir('kubernetes/'){
                        sh '''
                        helmversion=$(helm show chart myapp | grep version | cut -d ":" -f 2 | tr -d " ")
                        tar -czvf myapp-${helmversion}.tgz myapp/
                        curl -u $nexususername:$usernamepasswd http://13.126.96.40:8081/repository/helmrepo/ --upload-file myapp-${helmversion}.tgz -v
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