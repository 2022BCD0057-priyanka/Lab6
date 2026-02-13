pipeline {
    agent any

    environment {
        BEST_ACCURACY = credentials('best-accuracy')
        IMAGE_NAME = "2022bcd0057priyanka/wine_predict_2022bcd0057"
    }

    stages {

        // Stage 1: Checkout
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // Stage 2: Setup Python Virtual Environment
        stage('Setup Python Virtual Environment') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }

        // Stage 3: Train Model
        stage('Train Model') {
            steps {
                sh '''
                . venv/bin/activate
                python script/train.py
                '''
            }
        }

        // Stage 4: Read Accuracy
        stage('Read Accuracy') {
            steps {
                script {
                    def text = readFile('app/artifacts/metrics.json')
                    def json = new groovy.json.JsonSlurper().parseText(text)
                    env.CURRENT_ACCURACY = json.R2_Score.toString()
                    echo "Current Accuracy: ${env.CURRENT_ACCURACY}"
                }
            }
        }

        // Stage 5: Compare Accuracy
        stage('Compare Accuracy') {
            steps {
                script {
                    if (env.CURRENT_ACCURACY.toFloat() > env.BEST_ACCURACY.toFloat()) {
                        env.BUILD_IMAGE = "true"
                        echo "New model is better."
                    } else {
                        env.BUILD_IMAGE = "false"
                        echo "Model did not improve."
                    }
                }
            }
        }

        // Stage 6: Build Docker Image (Conditional)
        stage('Build Docker Image') {
            when {
                expression { env.BUILD_IMAGE == "true" }
            }
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        // Stage 7: Push Docker Image (Conditional)
        stage('Push Docker Image') {
            when {
                expression { env.BUILD_IMAGE == "true" }
            }
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        def image = docker.image("${IMAGE_NAME}:${env.BUILD_NUMBER}")
                        image.push()
                        image.push("latest")
                    }
                }
            }
        }
    }

    // Task 5: Archive artifacts
    post {
        always {
            archiveArtifacts artifacts: 'app/artifacts/**', fingerprint: true
        }
    }
}
