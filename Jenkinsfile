pipeline {
    
    agent any
    
    stages {
        stage("Clone Code") {
            steps {
                echo "Git clone"
                git url: "https://github.com/aadirai02/django-notes-app.git", branch: "main"
            }
        }

        stage("Build & Test") {
            steps {
                sh "docker build . -t django-notes-app:latest"
            }
        }

        stage("Push to DockerHub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHub", 
                    passwordVariable: "dockerHubPass", 
                    usernameVariable: "dockerHubUser")]) {
                    
                    sh "docker image tag django-notes-app:latest ${dockerHubUser}/django-notes-app:${BUILD_NUMBER}"
                    sh "docker login -u ${dockerHubUser} -p ${dockerHubPass}"
                    sh "docker push ${dockerHubUser}/django-notes-app:${BUILD_NUMBER}"
                }
            }
        }

        stage("Deploy") {
            steps {
                sh "docker compose up -d"
            }
        }
    }
}
