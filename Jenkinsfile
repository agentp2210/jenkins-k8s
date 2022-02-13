pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "agentp2210/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'npm install'
                sh 'npm test'
                // sh 'mkdir -p dist'
                // sh 'zip -r dist/trainSchedule.zip ./*'
                // archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    app = docker.build("agentp2210/train-schedule")
                    app.inside {
                        // sh 'echo $(curl localhost:8080)'
                        sh 'echo Hello World'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'main'
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
        stage('DeployToProduction') {
            when {
                branch 'main'
            }
            agent {
                label 'k8s'
            }
            steps {
                input 'Does the staging environment look OK?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true      // Allow k8s plugin to replace the variable in the train-schedule-kube.yml with the actual values
                )
            }
        }
    }
}