pipeline {
    agent none

    environment {
        IMAGE     = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
        CONTAINER = "${env.BRANCH_NAME == 'main' ? 'nodemain'      : 'nodedev'}"
        HOST_PORT = "${env.BRANCH_NAME == 'main' ? '3000'          : '3001'}"
    }

    stages {

        stage('Lint Dockerfile') {
            agent any
            steps {
                sh 'docker run --rm -i hadolint/hadolint hadolint --failure-threshold error - < Dockerfile'
            }
        }

        stage('Build & Test') {
            agent { docker { image 'node:16' } }
            stages {
                stage('Build') {
                    steps { sh 'npm install' }
                }
                stage('Test') {
                    steps { sh 'CI=true npm test' }
                }
            }
        }

        stage('Docker Build') {
            agent any
            steps {
                sh "docker build -t ${IMAGE} ."
            }
        }

        stage('Scan image (Trivy)') {
            agent any
            steps {
                sh """
                    docker run --rm \
                      -v /var/run/docker.sock:/var/run/docker.sock \
                      -v trivy-cache:/root/.cache/ \
                      aquasec/trivy image --severity HIGH,CRITICAL --exit-code 0 ${IMAGE}
                """
            }
        }

        stage('Deploy') {
            agent any
            steps {
                sh """
                    docker rm -f ${CONTAINER} || true
                    docker run -d --name ${CONTAINER} --expose ${HOST_PORT} -p ${HOST_PORT}:3000 ${IMAGE}
                """
            }
        }
    }
}