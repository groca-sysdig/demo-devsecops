pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = "grocamador/demo-scan"
        }
    
    stages {

    stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
            }
        }
     stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo 'Building docker image'
                sh "docker build -t grocamador/demo-scan:${env.BUILD_NUMBER} ."
            }
        }
        
      stage('Scanning Image with Sysdig') {
        steps {
            
            sh "echo grocamador/demo-scan${env.BUILD_NUMBER} > sysdig_secure_images"
            sysdig engineCredentialsId: 'sysdig-secure-api-credentials', name: 'sysdig_secure_images', inlineScanning: true
        }
       }  
        
            stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    
                    }
                }
            }
        }

        stage('Deploy to stage') {
            when {
                branch 'master'
            }
            steps {             
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-stage.yml',
                    enableConfigSubstitution: true
                )
 
            }
        }
        
       stage("Deploy to Production"){
            when {
                branch 'master'
            }
             steps {              
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
                
             }
         }
    }
}
