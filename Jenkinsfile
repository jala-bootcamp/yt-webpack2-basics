pipeline {
  // agent { label 'ubuntu-node-01' }
  agent any

  options {
    buildDiscarder(logRotator(numToKeepStr: '1', artifactNumToKeepStr: '1'))
  }

  triggers {
    pollSCM('0 */1 * * 1-5')
  }

  environment {
    NEXUS_REGISTRY_URL = 'http://192.168.1.65:8081/repository/npm-group/'
    NEXUS_AUTH_TOKEN   = credentials('nexus-auth-token')
  }

  stages {

    stage('Build') {
      steps {
        checkout scm
        withGradle {
          sh '''
            ./gradlew registrySetup \
                      nodeSetup \
                      npmInstall \
                      npm_run_build \
                      -PregistryUrl=$NEXUS_REGISTRY_URL \
                      -PauthToken=$NEXUS_AUTH_TOKEN
          '''
        }
      }
    }

    stage('Publish') {
      steps {
        withGradle {
          sh '''
            ./gradlew npm_publish
          '''
        }
      }
    }

    stage('Deploy') {
      node { label 'fedora-node-02' }
      when {
        branch 'main'
      }
      steps {
        sh 'curl http://rundeck.mio.com/execute/ae3824b4-e337-4f6c-b305-4a740a8b93c8'
      }
    }
  }
}
//
