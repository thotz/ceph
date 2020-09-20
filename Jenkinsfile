#!/usr/bin/env groovy

pipeline {
  agent {
    label 'huge && bionic && x86_64 && !xenial && !trusty'
  }

  environment {
    NPROC = "0"
  }

  stages {
    stage('Prepare') {
      steps {
        sh '''#!/usr/bin/env bash
        set -exo pipefail
        source ./run-make-check.sh
        FOR_MAKE_CHECK=1 prepare
        '''
      }
    }

    stage('Build') {
      steps {
        sh '''#!/usr/bin/env bash
        set -exo pipefail
        source ./run-make-check.sh
        configure
        build tests
        '''
      }
    }

    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            sh '''#!/usr/bin/env bash
            set -exo pipefail
            source ./run-make-check.sh
            cd build
            export CHECK_MAKEOPTS="$DEFAULT_MAKEOPTS --no-compress-output -T Test"
            run
            '''
          }
          post {
            always{
              xunit(
                [CTest(
                  deleteOutputFiles: true,
                  failIfNotNew: true,
                  pattern: 'build/Testing/**/Test.xml',
                  skipNoTestFiles: true,
                  stopProcessingIfError: true)
                ]
              )
            }
          }
        }

        stage('API Tests') {
          // Disabled so far
          when { expression { false } }

          steps {
            sh '''#!/usr/bin/env bash
            set -exo pipefail
            cd src/pybind/mgr/dashboard
            ./install-backend-api-test-deps.sh
            '''

            sh '''#!/usr/bin/env bash
            set -exo pipefail
            cd src/pybind/mgr/dashboard
            ./run-backend-api-tests.sh
            '''
          }
        }
      } // parallel
    } // stage ('Test')

  } // stages
}
