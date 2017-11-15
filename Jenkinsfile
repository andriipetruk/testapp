env.DOCKERHUB_USERNAME = 'andriipetruk'
env.APP_NAME = 'testapp'

 node("docker") {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
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
