
env.DOCKERHUB_USERNAME = 'andriipetruk'
env.APP_NAME = 'testapp'

  node("docker") {
    checkout scm

    stage("Integration") {
         checkout scm
         stage("Unit Test") {
             sh "docker run --rm -w /code -v ${WORKSPACE}:/code mhart/alpine-node sh -c \"npm i; npm test\""
             }
             
        stage("Integration Test") {
            try {
                sh "docker build -t ${APP_NAME} ."
                sh "docker rm -f ${APP_NAME} || true"
                sh "docker run -d -p 8081:8080 --name=${APP_NAME} ${APP_NAME}"
                // env variable is used to set the server where go test will connect to run the test
                //sh "docker run --rm -v ${WORKSPACE}:/code --link=jenkins_node -e SERVER=jenkins_node mhart/alpine-node npm test"
                }
                catch(e) {
                    error "Integration Test failed"
                    }
                finally {
                        sh "docker rm -f ${APP_NAME} || true"
                        sh "docker ps -aq | xargs docker rm || true"
                        sh "docker images -aq -f dangling=true | xargs docker rmi || true"
                    }
                }
        stage("Build") { sh "docker build -t ${DOCKERHUB_USERNAME}/jenkins_node:${BUILD_NUMBER} ." }
        stage("Publish") { withDockerRegistry([credentialsId: 'docker-hub-credentials']) {
            sh "docker push ${DOCKERHUB_USERNAME}/jenkins_node:${BUILD_NUMBER}"
            }
            }
    }
  }

  
