pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "systemctl/train-schedule"
    }
    stages {
        stage('Build Gradle') {
            when {
                branch 'k8stest'
            }
            steps {
                echo 'Running build automation'
                sh 'pwd && ls -ltr && ls dist/'
                sh './gradlew build --no-daemon'
                sh 'ls -ltr'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'k8stest'
            }
            steps {
                input 'Build Docker Image?'
                milestone(1)
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
                branch 'k8stest'
            }
            steps {
                input 'Push Docker Image?'
                milestone(1)
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
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kuberspl',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
