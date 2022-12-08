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
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh 'mvn clean install sonar:sonar'
                    }
                }
            }
        }
        stage('Quality gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                }
            }
        }
        stage('nexus artifact uploader'){
            steps{
                script{
                    def readpom = readMavenPom file: 'pom.xml'
                    def readversion = readpom.version
                    def readrepo = readversion.endsWith("SNAPSHOT") ? "casioproject-snapshot" : "casioproject-release"
                    nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'nexus-creds', groupId: 'com.javatechie', nexusUrl: '13.126.234.16:8081', nexusVersion: 'nexus3', protocol: 'http', repository: readrepo, version: readversion

                }
            }
        }
        stage('docker build image'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', passwordVariable: 'dpassword', usernameVariable: 'dusername')]) {
                        sh 'docker login -u $dusername -p $dpassword'
                        sh 'docker build -t $JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dusername/$JOB_NAME:latest'
                    }
                }
            }
        }
        stage('docker push'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', passwordVariable: 'dpassword', usernameVariable: 'dusername')]) {
                        sh 'docker login -u $dusername -p $dpassword'
                        sh 'docker push $dusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker push $dusername/$JOB_NAME:latest'
                        sh 'docker rmi -f $dusername/$JOB_NAME:latest'
                        sh 'docker rmi -f $dusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker rmi -f $JOB_NAME:v1.$BUILD_ID'
                    }
                }
            }
        }
        stage('installing helm datree'){
            steps{
                script{
                    sh 'ansible-playbook ansible.yml'
                }
            }
        }
        stage('misconfigs using datree'){
            steps{
                script{
                    dir('kubernetes/') {
                       withEnv(['DATREE_TOKEN=22ecd219-bce0-4cb8-8a9a-efab1589ab1d']){
                           sh 'helm datree test myapp/'
                       }
                        

                    } 
                }
            }
        }
        stage('nexus helm chart uploader'){
            steps{
                script{
                    dir('kubernetes/') {
                        withCredentials([usernamePassword(credentialsId: 'nexushelm', passwordVariable: 'npassword', usernameVariable: 'nusername')]) {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d ":" -f 2 | tr -d " ")
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u $nusername:$npassword http://13.126.234.16:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }
                    }
                }
            }
        }
    }
}