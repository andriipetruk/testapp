env.DOCKERHUB_USERNAME = 'andriipetruk'
env.APP_NAME = 'testapp'

 node("docker") {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

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
      }finally {
        sh "docker rm -f ${APP_NAME} || true"
        sh "docker ps -aq | xargs docker rm || true"
        sh "docker images -aq -f dangling=true | xargs docker rmi || true"
      }
    }

    stage("Build image") {
     sh "docker build -t ${DOCKERHUB_USERNAME}/${APP_NAME}:${BUILD_NUMBER} ."
    }
    
    stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

      //  app.inside {
      //      sh 'echo "Tests passed"'
      //  }
    }


  
    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
           // app.push("${env.BUILD_NUMBER}")
           // app.push("latest")
         sh "docker push ${DOCKERHUB_USERNAME}/${APP_NAME}:${BUILD_NUMBER}"
        }
     
    }
}
