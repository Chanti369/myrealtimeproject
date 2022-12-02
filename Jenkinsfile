pipeline{
    parameters{
        choice(name: 'action', choices: 'create\ndestroy\ndestroycluster', description: 'create,destroy the cluster')
        string(name: 'kopscluster', defaultValue: 'devcluster', description: 'which environment')
        string(name: 'updaterepo', defaultValue: 'bitnami', description: 'name the repo you want to update')
        string(name: 'releasename', defaultValue: 'my-release', description: 'name your release')
        string(name: 'chartname', defaultValue: 'jenkins', description: 'name of the chart that you want install')
    }
    agent{
        node{
            label "${params.kopscluster}"
        }
    }
    stages{
        stage('git checkout'){
            steps{
                script{
                    git branch: 'main', url: 'https://github.com/Chanti369/myrealtimeproject.git'
                }
            }
        }
        stage('kops cluster'){
            when { expression {params.action == 'create'}}
            steps{
                script{
                    sh 'helm repo update '${params.updaterepo}''
                    sh 'helm install '${params.releasename}' '${params.updaterepo}'/'${params.chartname}''
                }
            }
        }
        stage('helm uninstall'){
            when { expression {params.action == 'destroy'}}
            steps{
                script{
                    sh 'helm uninstall '${params.releasename}''
                }
            }
        }
    }
}