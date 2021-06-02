#!groovy

// def appNameVar
// def appVerVar

pipeline {
    environment { 
        registry = "santhoshv/boutique-frontend" 
        registryCredential = 'dockerhub_id' 
        dockerImage = '' 
        dockerVersion = ":$BUILD_NUMBER"+".1.0"
    }
    agent any
    stages {
        // stage('Cloning our Git') { 
        //     steps { 
        //         git 'https://github.com/SanthoshVajinepalli/frontend-demo.git' 
        //     }
        // } 
        stage('Building image') { 
            steps { 
                script { 
                    dockerImage = docker.build(registry + dockerVersion) 
                }
            } 
        }
        stage('Push image') { 
            steps { 
                script { 
                    docker.withRegistry( '', registryCredential ) { 
                        dockerImage.push() 
                    }
                } 
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                bat '''
                set KUBECONFIG=demo.kubeconfig.yaml
                set KUBECTL_VSPHERE_PASSWORD=1234Qwer$
                kubectl vsphere login --server=https://192.168.0.72 --vsphere-username=demo@vsphere.local --tanzu-kubernetes-cluster-namespace=demo --tanzu-kubernetes-cluster-name=demo-cluster --insecure-skip-tls-verify
                kubectl config use-context demo-cluster
                
                helm uninstall frontend
                helm install  frontend -f frontend/env/dev-values.yaml --version 12.1.0 frontend/
                '''
            }
        }
    }
}