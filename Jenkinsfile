
env.DOCKERHUB_USERNAME = 'andriipetruk'
env.APP_NAME = 'testapp'

  node("docker") {
    checkout scm

    stage("Staging") {
      try {
        sh "docker rm -f jenkins_node || true"
        sh "docker run -d -p 8080:8080 --name=jenkins_node ${DOCKERHUB_USERNAME}/jenkins_node:${BUILD_NUMBER}"
        sh "docker run --rm ${DOCKERHUB_USERNAME}/jenkins_node:${BUILD_NUMBER} npm test"

      } catch(e) {
        error "Staging failed"
      } finally {
        sh "docker rm -f jenkins_node || true"
        sh "docker ps -aq | xargs docker rm || true"
        sh "docker images -aq -f dangling=true | xargs docker rmi || true"
      }
    }
  }

  node("docker") {
    stage("Production") {
      try {
        // Create the service if it doesn't exist otherwise just update the image
        sh '''
          SERVICES=$(docker service ls --filter name=jenkins_node --quiet | wc -l)
          if [[ "$SERVICES" -eq 0 ]]; then
            docker network rm jenkins_node || true
            docker network create --driver overlay --attachable jenkins_node
            docker service create --replicas 3 --network jenkins_node --name jenkins_node -p 8080:8080 ${DOCKERHUB_USERNAME}/jenkins_node:${BUILD_NUMBER}
          else
            docker service update --image ${DOCKERHUB_USERNAME}/jenkins_node:${BUILD_NUMBER} jenkins_node
          fi
          '''
        // run some final tests in production
        checkout scm
        sh '''
          sleep 60s 
          for i in `seq 1 20`;
          do
            STATUS=$(docker service inspect --format '{{ .UpdateStatus.State }}' jenkins_node)
            if [[ "$STATUS" != "updating" ]]; then
              docker run --rm --network jenkins_node ${DOCKERHUB_USERNAME}/jenkins_node:${BUILD_NUMBER} npm test
              break
            fi
            sleep 10s
          done
          
        '''
      }catch(e) {
        sh "docker service update --rollback  jenkins_node"
        error "Service update failed in production"
      }finally {
        sh "docker ps -aq | xargs docker rm || true"
      }
    }
  }
