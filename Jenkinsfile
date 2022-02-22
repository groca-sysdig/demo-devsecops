pipeline {
    agent any
    
    environment {
        //be sure to replace "grocamador" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "grocamador/train-schedule"
        }
    
    stages {

        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
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
                    
                    }
                }
            }
        }

        stage('Push Docker Image to latest') {
            when {
                branch 'master'
            }
            steps {
                script {
                    
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("latest")
                    }
                }
            }
        }
        stage('Clean') {
        steps{
            echo 'cleaning up artifacts'
                script{
                try{
                sh 'rm train-schedule.tar'
                echo 'Image deleted'
                } catch (Exception e) {
                echo 'Already deleted'  
                }
                }
            }
        }
        stage('Deploy to stage') {
            when {
                branch 'master'
            }
            steps {
            sh ("""     
                  kubectl delete -f train-schedule-kube-stage.yml
                  kubectl apply -f train-schedule-kube-stage.yml
                """)
 
                // Other tentative that didnt work:
 //            kubectl set image deployment/train-schedule-deployment-stage train-schedule-stage=grocamador/train-schedule
 //            kubectl set image deployment/train-schedule-deployment-stage train-schedule-stage=grocamador/train-schedule:latest
            }
        }
        
       stage("Deploy to Production"){
            when {
                branch 'master'
            }
             steps {              
                input 'Deploy to Production?'
                milestone(1)
//              With KUBECTL and Kubeconfig       
              sh ("""                
                  echo \$KUBECONFIG
                  kubectl delete -f train-schedule-kube.yml
                  kubectl apply -f train-schedule-kube.yml
                """)

                 
//                 kubernetesDeploy(
//                    kubeconfigId: 'kubeconfig',
//                    configs: 'deploy.yml',
//                    enableConfigSubstitution: true
//                )

                
             }
         }
    }
}
