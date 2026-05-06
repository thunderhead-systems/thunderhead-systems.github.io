pipeline {
  agent any

  environment {
    NEXUS_URL = 'https://nexus.softsurve.com'
  }

  stages {

    stage('Pre-flight') {
      steps {
        script {
          def msg = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
          if (msg.contains('[skip ci]')) {
            currentBuild.result = 'NOT_BUILT'
            error('Commit contains [skip ci] — aborting.')
          }
        }
      }
    }

    stage('Build') {
      agent {
        docker {
          image 'ruby:3.1-slim'
          reuseNode true
        }
      }
      steps {
        sh '''
          apt-get update -qq && apt-get install -y -qq build-essential
          bundle install --path vendor/bundle
          bundle exec jekyll build
        '''
      }
      post {
        success {
          archiveArtifacts artifacts: '_site/**', allowEmptyArchive: false
        }
      }
    }

  }

  post {
    always { cleanWs() }
  }
}
