pipeline{
    parameters{
        choice(name: 'action', choices: 'create\ndestroy\ndestroycluster', description: 'create,destroy the cluster')
        string(name: 'kopscluster', defaultValue: 'devcluster', description: 'which environment')
    }
    agent{
        node{
            label ${params.kopscluster}
        }
    }
    stages{
        stage('kops cluster'){
            when { expression {params.action == 'create'}}
            steps{
                script{
                    sh 'helm repo update bitnami'
                    sh 'helm install my-release bitnami/jenkins'
                }
            }
        }
        stage('helm uninstall'){
            when { expression {params.action == 'destroy'}}
            steps{
                script{
                    sh 'helm uninstall my-release'
                }
            }
        }
    }
}