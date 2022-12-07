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
        stage('install helm datree plugin'){
            steps{
                script{
                    sh 'ansible-playbook /home/ec2-user/workspace/project2/ansible.yml'
                }
            }
        }
        stage('datree'){
            steps{
                script{
                    dir('kubernetes/') {
                      withEnv(['DATREE_TOKEN=22ecd219-bce0-4cb8-8a9a-efab1589ab1d']) {
                        sh 'helm datree test myapp'
                      }
                    }  
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
                    def readrepo = readversion.endsWith("SNAPSHOT") ? "project1-snapshot" : "project1-release"
                    nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'nexus-creds', groupId: 'com.javatechie', nexusUrl: '65.2.124.211:8081', nexusVersion: 'nexus3', protocol: 'http', repository: readrepo, version: readversion
                }
            }
        }
        stage('docker build image'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                        sh 'docker login -u $dockerusername -p $dockerpassword'
                        sh 'docker build -t $JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername/$JOB_NAME:latest'
                        sh 'docker rmi -f $JOB_NAME:v1.$BUILD_ID'
                    }
                }
            }
        }
        stage('docker push images'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                      sh 'docker login -u $dockerusername -p $dockerpassword'
                      sh 'docker push $dockerusername/$JOB_NAME:v1.$BUILD_ID' 
                      sh 'docker push $dockerusername/$JOB_NAME:latest' 
                      sh 'docker rmi -f $dockerusername/$JOB_NAME:latest'
                      sh 'docker rmi -f $dockerusername/$JOB_NAME:v1.$BUILD_ID'

                    }
                }
            }
        }
        stage('indentifying misconfigs using datree') {
            steps{
                script{
                    dir('kubernetes/myapp/') {
                        withEnv(['DATREE_TOKEN=22ecd219-bce0-4cb8-8a9a-efab1589ab1d']) {
                            sh 'helm datree test *'
                        }
                    }
                }
            }
        }
        stage('sending helm chart to nexus repo'){
            steps{
                script{
                    dir('kubernetes/') {
                        withCredentials([usernamePassword(credentialsId: 'nexus-credentials', passwordVariable: 'nexuspassword', usernameVariable: 'nexususername')]) {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d ":" -f 2 | tr -d " ")
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u $nexususername:$nexuspassword http://65.2.124.211:8081/repository/helmrepo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }    

                    }
                }
            }
        }
    }
}