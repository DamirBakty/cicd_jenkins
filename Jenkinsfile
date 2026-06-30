 pipeline {
      agent any

      tools {
          nodejs 'node'
      }

      environment {
          IMAGE     = "${env.BRANCH_NAME == 'main' ? 'nodemain:v1.0' : 'nodedev:v1.0'}"
          CONTAINER = "${env.BRANCH_NAME == 'main' ? 'nodemain'      : 'nodedev'}"
          HOST_PORT = "${env.BRANCH_NAME == 'main' ? '3000'          : '3001'}"
      }

      stages {
          stage('Build') {
              steps {
                  sh 'npm install'
              }
          }
          stage('Test') {
              steps {
                  sh 'CI=true npm test'
              }
          }
          stage('Docker Build') {
              steps {
                  sh "docker build -t ${IMAGE} ."
              }
          }
          stage('Deploy') {
              steps {
                  sh """
                      docker rm -f ${CONTAINER} || true
                      docker run -d --name ${CONTAINER} --expose ${HOST_PORT} -p ${HOST_PORT}:3000 ${IMAGE}
                  """
              }
          }
      }
  }