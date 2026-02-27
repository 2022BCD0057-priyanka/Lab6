pipeline {
    agent any

    environment {
        IMAGE = "2022bcd0057priyanka/wine_predict_2022bcd0057:8"
        CONTAINER = "wine_validation_container"
        PORT = "8000"
    }

    stages {

        stage('Check Docker') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Pull Docker Image') {
            steps {
                sh "docker pull ${IMAGE}"
            }
        }

        stage('Run Container') {
            steps {
                sh "docker run -d -p ${PORT}:8000 --name ${CONTAINER} ${IMAGE}"
            }
        }

        stage('Wait for API Readiness') {
            steps {
                script {
                    echo "Waiting for API to start..."
                    sleep 15
                }
            }
        }

        stage('Send Valid Inference Request') {
            steps {
                script {
                    def response = sh(
                        script: """curl -s -X POST "http://localhost:${PORT}/predict?fixed_acidity=7.4&volatile_acidity=0.7&citric_acid=0.0&residual_sugar=1.9&chlorides=0.076&free_sulfur_dioxide=11&total_sulfur_dioxide=34&density=0.9978&pH=3.51&sulphates=0.56&alcohol=9.4" """,
                        returnStdout: true
                    ).trim()

                    echo "Valid API Response: ${response}"

                    if (!response.contains("wine_quality")) {
                        error("Prediction field 'wine_quality' missing!")
                    }
                }
            }
        }

        stage('Send Invalid Request') {
            steps {
                script {
                    def status = sh(
                        script: """curl -s -o /dev/null -w "%{http_code}" -X POST "http://localhost:${PORT}/predict?fixed_acidity=7.4" """,
                        returnStdout: true
                    ).trim()

                    echo "Invalid Request Status Code: ${status}"

                    if (status == "200") {
                        error("Invalid input should not return 200!")
                    }
                }
            }
        }

        stage('Stop and Remove Container') {
            steps {
                sh "docker stop ${CONTAINER} || true"
                sh "docker rm ${CONTAINER} || true"
            }
        }
    }

    post {
        success {
            echo "🎉 PIPELINE PASSED: Model validation successful!"
        }
        failure {
            echo "❌ PIPELINE FAILED: Validation error detected!"
        }
        always {
            sh "docker rm -f ${CONTAINER} || true"
        }
    }
}