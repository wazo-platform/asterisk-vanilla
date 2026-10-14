def packageName = null
def version = null
def isRc = false

pipeline {
  agent any
  triggers {
    githubPush()
    pollSCM('H H * * *')
  }
  environment {
    MAIL_RECIPIENTS = 'dev+tests-reports@wazo.community'
  }
  options {
    skipStagesAfterUnstable()
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
  }
  stages {
    stage ('Prepare') {
      steps {
        script {
          isRc = env.GIT_BRANCH == 'origin/asterisk-rc'
          packageName = sh(script: 'dpkg-parsechangelog --show-field source', returnStdout: true).trim()
          version = sh(script: 'dpkg-parsechangelog --show-field version', returnStdout: true).trim()
          if (isRc) {
            version = "${version}~astrc"
          }
          currentBuild.displayName = "${JOB_NAME} ${version}"
          currentBuild.description = "Build Debian package ${packageName} ${version}"
        }
      }
    }
    stage('Debian build and deploy') {
      steps {
        script {
          def parameters = [
            string(name: 'PACKAGE', value: packageName),
            string(name: 'VERSION', value: version),
          ]
          if (isRc) {
            parameters += [
              string(name: 'BRANCH', value: 'asterisk-rc'),
              string(name: 'DISTRIBUTION', value: 'asterisk-rc'),
            ]
          }
          build job: 'build-package-multi-arch', parameters: parameters
        }
      }
    }
  }
  post {
    failure {
      emailext to: "${MAIL_RECIPIENTS}", subject: '${DEFAULT_SUBJECT}', body: '${DEFAULT_CONTENT}'
    }
    fixed {
      emailext to: "${MAIL_RECIPIENTS}", subject: '${DEFAULT_SUBJECT}', body: '${DEFAULT_CONTENT}'
    }
  }
}
