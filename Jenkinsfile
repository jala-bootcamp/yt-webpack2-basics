pipeline {
  // agent { label 'ubuntu-node-01' }
  agent any

  options {
    buildDiscarder(logRotator(daysToKeepStr: '5', numToKeepStr: '5', artifactNumToKeepStr: '5'))
    skipDefaultCheckout(true)
  }

  triggers {
    pollSCM('0 */1 * * 1-5')
  }

  environment {
    NEXUS_REGISTRY_URL = 'http://192.168.1.65:8081/repository/npm-group/'
    NEXUS_AUTH_TOKEN   = credentials('nexus-auth-token')
  }

  stages {

    stage('Pull code') {
      steps {
        cleanWs()
        checkout scm
      }
    }

    stage('Build') {
      steps {
        withGradle {
          sh '''
            ./gradlew -is --no-daemon \
                      registrySetup \
                      nodeSetup \
                      npmInstall \
                      npm_run_build \
                      -PregistryUrl=$NEXUS_REGISTRY_URL \
                      -PauthToken=$NEXUS_AUTH_TOKEN
          '''
        }
      }
    }

    // stage('Test') {
    //   steps {
    //     withGradle {
    //       sh './gradlew npm_test'
    //     }
    //   }
    // }

    stage('Publish') {
      steps {
        withGradle {
          sh '''
            ./gradlew -is npm_publish --no-daemon
          '''
        }
      }
    }

    stage('Deploy') {
      agent {
        label 'fedora-node-02'
      }
      when {
        branch 'main'
      }
      steps {
          echo 'test in node'
      }
    }
  }
  post {
        // Clean after build
        always {
            cleanWs(cleanWhenNotBuilt: false,
                    deleteDirs: true,
                    disableDeferredWipeout: true,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                               [pattern: '.propsfile', type: 'EXCLUDE']])
        }
  }
}
//
