#!/usr/bin/env groovy
 @Library('github.com/zanata/zanata-pipeline-library@master')

/* Only keep the 10 most recent builds. */
def projectProperties = [
  [
    $class: 'BuildDiscarderProperty',
    strategy: [$class: 'LogRotator', numToKeepStr: '10']
  ],
]

properties(projectProperties)

def surefireTestReports='target/surefire-reports/TEST-*.xml'

timestamps {
  node {
    ansicolor {
       try {
         pullRequests.ensureJobDescription()
         stage('Checkout') {
          info.printNode()
          notify.started()
          checkout scm
        }

        stage('Check build tools') {
          info.printNode()

          // Note: if next stage happens on another node, mvnw might have to download again
          sh "./mvnw --version"
        }

        stage('Build') {
          info.printNode()
          info.printEnv()

          // globstar might failed to match
          sh "find . -path \"*/${surefireTestReports}\" -delete"

          sh """./mvnw -e clean verify \
                     --batch-mode \
                     --settings .travis-settings.xml \
                     --update-snapshots \
                     -Dmaven.test.failure.ignore \
                     -DstaticAnalysis \
             """
             notify.successful()
             notify.successful()
        }
      } catch (e) {
         notify.failed()
         currentBuild.result='FAILURE'
         throw e
      } finally {
        junit allowEmptyResults: true,
              keepLongStdio: true,
              testDataPublishers: [[$class: 'StabilityTestDataPublisher']],
              testResults: "**/${surefireTestReports}"

        step([ $class: 'JacocoPublisher' ])
      }
    }
  }
}
