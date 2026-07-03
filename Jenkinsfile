pipeline {
    agent none

    environment {
        IMAGE     = "${env.BRANCH_NAME == 'main' ? 'ultimate1465/nodemain:v1.0' : 'ultimate1465/nodedev:v1.0'}"
        CONTAINER = "${env.BRANCH_NAME == 'main' ? 'nodemain'                   : 'nodedev'}"
        HOST_PORT = "${env.BRANCH_NAME == 'main' ? '3000'                       : '3001'}"
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

        stage('Push to Docker Hub') {
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                 usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                    sh "docker push ${IMAGE}"
                }
            }
        }

        stage('Trigger deploy') {
            agent any
            steps {
                script {
                    def deployJob = (env.BRANCH_NAME == 'main') ? 'Deploy_to_main' : 'Deploy_to_dev'
                    build job: deployJob, wait: false
                }
            }
        }
    }
}
