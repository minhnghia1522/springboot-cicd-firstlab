pipeline {

  agent any

  environment {
    MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
  }
  stages {
    stage('Check version docker') {
      steps {
          sh 'docker --version'
      }
    }
    stage('Packaging/Pushing imagae') {
      steps {
        // This step should not normally be used in your script. Consult the inline help for details.
        withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
          sh 'docker build -t minhnghia22/springboot .'
          sh 'docker push minhnghia22/springboot'
        }
      }
    }

    stage('Deploy MySQL to DEV') {
      steps {
        echo 'Deploying and cleaning'
        sh 'docker image pull mysql:8.0'
        sh 'docker network create dev || echo "this network exists"'
        sh 'docker container stop n-mysql || echo "this container does not exist" '
        sh 'echo y | docker container prune '
        sh 'docker volume rm n-mysql-data || echo "no volume"'

        sh "docker run --name n-mysql --rm --network dev -v n-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
        sh 'sleep 120'
        sh "docker exec -i n-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
      }
    }

    stage('Deploy Spring Boot to DEV') {
      steps {
        echo 'Deploying and cleaning'
        sh 'docker image pull minhnghia22/springboot'
        sh 'docker container stop nghia-springboot || echo "this container does not exist" '
        sh 'docker network create dev || echo "this network exists"'
        sh 'echo y | docker container prune '

        sh 'docker container run -d --rm --name nghia-springboot -p 8081:8080 --network dev minhnghia22/springboot'
      }
    }

  }
  post {
    // Clean after build
    always {
      cleanWs()
    }
  }
}