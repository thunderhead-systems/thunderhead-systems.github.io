pipeline {
  // Pinned, not `agent any`. An unpinned agent can land on the macOS
  // (saturn) or Windows (amalthea) node, neither of which has a docker
  // CLI, and every docker stage below then dies with "docker: not found".
  // Only europa and callisto carry `linux-build`. (CI hardening 2026-09-13)
  agent { label 'linux-build' }

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
          // `-u root` is required: the build step apt-get installs
          // build-essential so native gem extensions (eventmachine,
          // http_parser.rb, json) can compile. As the default container
          // user apt fails with
          //   E: List directory /var/lib/apt/lists/partial is missing. (13: Permission denied)
          // and because that line is `apt-get update && apt-get install`
          // with no `set -e`, the step carries on and dies confusingly at
          //   make failed  No such file or directory - make
          // (CI hardening 2026-09-18)
          args '-u root'
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
