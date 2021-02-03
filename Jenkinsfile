pipeline {
  // agent { label 'ubuntu-node-01' }
  agent any

  triggers {
    pollSCM('0 */1 * * 1-5')
  }

  stages {

    stage('Build') {
      steps {
        checkout scm
        withGradle {
          sh './gradlew registrySetup nodeSetup npmInstall npm_run_build'
        }
      }
    }

    stage('Publish') {
      steps {
        withGradle {
          sh './gradlew npm_publish'
        }
      }
    }
    
    stage('Deploy') {
      when {
        branch 'release'
      }
      steps {
        sh 'curl http://rundeck.mio.com/execute/ae3824b4-e337-4f6c-b305-4a740a8b93c8'
      }
    }
  }
}
//
