pipeline {
    agent any

    environment {
        BEST_ACCURACY = credentials('best-accuracy')
        IMAGE_NAME = "2022bcd0057priyanka/wine_predict_2022bcd0057"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python Virtual Environment') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }

        stage('Train Model') {
            steps {
                sh '''
                . venv/bin/activate
                python script/train.py
                '''
            }
        }

        stage('Read Accuracy') {
            steps {
                script {
                    def metrics = readJSON file: 'output/metrics.json'
                    env.CURRENT_ACCURACY = metrics.R2_Score.toString()
                }
            }
        }

        stage('Compare Accuracy') {
            steps {
                script {
                    if (env.CURRENT_ACCURACY.toFloat() > env.BEST_ACCURACY.toFloat()) {
                        env.BUILD_IMAGE = "true"
                    } else {
                        env.BUILD_IMAGE = "false"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { env.BUILD_IMAGE == "true" }
            }
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        def image = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
                        image.push()
                        image.push("latest")
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'output/**', fingerprint: true
        }
    }
}
