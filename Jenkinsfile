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
                    echo "Waiting for API..."
                    sleep 15
                }
            }
        }

        stage('Send Valid Inference Request') {
            steps {
                script {

                    def httpStatus = sh(
                        script: """curl -s -o response.json -w "%{http_code}" -X POST "http://host.docker.internal:${PORT}/predict?fixed_acidity=7.4&volatile_acidity=0.7&citric_acid=0.0&residual_sugar=1.9&chlorides=0.076&free_sulfur_dioxide=11&total_sulfur_dioxide=34&density=0.9978&pH=3.51&sulphates=0.56&alcohol=9.4" """,
                        returnStdout: true
                    ).trim()

                    def response = readFile('response.json')

                    echo "HTTP Status: ${httpStatus}"
                    echo "Valid API Response: ${response}"

                    if (httpStatus != "200") {
                        error("Valid request failed! HTTP Status: ${httpStatus}")
                    }

                    if (!response.contains("wine_quality")) {
                        error("Prediction field missing!")
                    }
                }
            }
        }

        stage('Send Invalid Request') {
            steps {
                script {

                    def invalidStatus = sh(
                        script: """curl -s -o /dev/null -w "%{http_code}" -X POST "http://host.docker.internal:${PORT}/predict?fixed_acidity=7.4" """,
                        returnStdout: true
                    ).trim()

                    echo "Invalid Status Code: ${invalidStatus}"

                    if (invalidStatus == "200") {
                        error("Invalid request should not return 200!")
                    }
                }
            }
        }

        stage('Stop Container') {
            steps {
                sh "docker stop ${CONTAINER} || true"
                sh "docker rm ${CONTAINER} || true"
            }
        }
    }

    post {
        success {
            echo "🎉 PIPELINE PASSED"
        }
        failure {
            echo "❌ PIPELINE FAILED"
        }
        always {
            sh "docker rm -f ${CONTAINER} || true"
        }
    }
}