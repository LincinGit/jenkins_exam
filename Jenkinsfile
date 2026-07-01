pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "lincin65" // replace this with your docker-id
DOCKER_IMAGE_CAST = "cast-service"
DOCKER_IMAGE_MOVIE = "movie-cast"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build

}
agent any // Jenkins will be able to select all available agents
stages {
  stage(' Docker Build Cast-Service'){ // docker build image stage
    steps {
      script {
      sh '''
        pwd
        ls -la
        find . -name Dockerfile
        docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG app/cast-service
      sleep 6
      '''
      }
    }
  }
  stage(' Docker Build Movie-Service'){ // docker build image stage
    steps {
      script {
      sh '''
        pwd
        ls -la
        find . -name Dockerfile
        docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG  app/movie-service
      sleep 6
      '''
      }
    }
  }  
  stage('Docker run Cast Container'){ // run container from our builded image
    steps {
      script {
      sh '''
      docker run -d -p 80:80 --name cast-service $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
      sleep 10
      '''
      }
    }
  }
  stage('Docker run Movie Container'){ // run container from our builded image
    steps {
      script {
      sh '''
      docker run -d -p 80:80 --name movie-service $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
      sleep 10
      '''
      }
    }
  }  
  stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
    steps {
      script {
      sh '''
      // curl localhost
      curl http://localhost:8081
      curl http://localhost:8082
      '''
      }
    }
  }
  stage('Docker Push'){ //we pass the built image to our docker hub account
    environment
    {
      DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
    }
    steps {
      script {
        sh '''
        // docker login -u $DOCKER_ID -p $DOCKER_PASS
        echo "$DOCKER_PASS" | docker login -u "$DOCKER_ID" --password-stdin
        docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
        docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
        '''
      }
    }
    // steps {
    //   script {
    //     sh '''
    //     docker login -u $DOCKER_ID -p $DOCKER_PASS
    //     docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
    //     '''
    //   }
    // }
  }
  stage('Deploy to dev') {
      environment {
          KUBECONFIG = credentials('config')
      }
      steps {
          sh '''
          mkdir -p .kube
          cp $KUBECONFIG .kube/config
          helm dependency update charts/umbrella
          helm upgrade --install jenkins_exam_app charts/umbrella \
              --namespace dev \
              --create-namespace \
              --set cast.image.repository=${DOCKER_ID}/${DOCKER_IMAGE_CAST} \
              --set cast.image.tag=${DOCKER_TAG} \
              --set movie.image.repository=${DOCKER_ID}/${DOCKER_IMAGE_MOVIE} \
              --set movie.image.tag=${DOCKER_TAG}             
          '''
      }
  }
  stage('Deploy to staging') {
      environment {
          KUBECONFIG = credentials('config')
      }
      steps {
          sh '''
          mkdir -p .kube
          cp $KUBECONFIG .kube/config
          helm dependency update charts/umbrella
          helm upgrade --install jenkins_exam_app charts/umbrella \
              --namespace staging \
              --create-namespace \
              --set cast.image.repository=${DOCKER_ID}/${DOCKER_IMAGE_CAST} \
              --set cast.image.tag=${DOCKER_TAG} \
              --set movie.image.repository=${DOCKER_ID}/${DOCKER_IMAGE_MOVIE} \
              --set movie.image.tag=${DOCKER_TAG}             
          '''
      }
  }
  stage('Deploy to prod') {
      environment {
          KUBECONFIG = credentials('config')
      }
      steps {
        timeout(time: 15, unit: "MINUTES") {
          input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
          mkdir -p .kube
          cp $KUBECONFIG .kube/config
          helm dependency update charts/umbrella
          helm upgrade --install jenkins_exam_app charts/umbrella \
              --namespace prod \
              --create-namespace \
              --set cast.image.repository=${DOCKER_ID}/${DOCKER_IMAGE_CAST} \
              --set cast.image.tag=${DOCKER_TAG} \
              --set movie.image.repository=${DOCKER_ID}/${DOCKER_IMAGE_MOVIE} \
              --set movie.image.tag=${DOCKER_TAG}             
          '''
      }
    }  
  }
  }
}
