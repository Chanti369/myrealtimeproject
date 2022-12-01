pipeline{
    agent any
    tools{
        maven "MAVEN"
    }
    stages{
        stage('Git checkout'){
            steps{
                script{
                    git branch: 'main', url: 'https://github.com/Chanti369/myrealtimeproject.git'
                }
            }
        }
        stage('static code analysis using sonarqube scanner'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube'){
                        sh 'mvn clean install sonar:sonar'
                    }
                }
            }
        }
        stage('Qualitygate'){
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
                    def readrepo = readversion.endsWith("SNAPSHOT") ? "project1-snapshot" : "project1-release"
                    nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'nexus', groupId: 'com.javatechie', nexusUrl: '65.0.177.197:8081', nexusVersion: 'nexus3', protocol: 'http', repository: readrepo, version: readversion
                }
            }
        }
        stage('docker build image'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'docker1', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername1')]) {
                        sh 'docker login -u $dockerusername1 -p $dockerpassword'
                        sh 'docker build -t $JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername1/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername1/$JOB_NAME:latest'
                    }
                }
            }
        }
        stage('docker push'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'docker1', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername1')]){
                        sh 'docker login -u $dockerusername1 -p $dockerpassword'
                        sh 'docker push $dockerusername1/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker push $dockerusername1/$JOB_NAME:latest'
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
        stage('pushing helm chart'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'nexushelm', passwordVariable: 'nexuspasswd', usernameVariable: 'nexusuname')]) {
                        dir('kubernetes/') {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d ":" -f 2 | tr -d " ")
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u $nexusuname:$nexuspasswd http://65.0.177.197:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
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