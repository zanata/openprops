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
      // ensure the build can handle at-signs in paths:
      dir("@") {
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
            sh """./mvnw -e clean verify \
                --batch-mode \
                --settings .travis-settings.xml \
                --update-snapshots \
                -Dmaven.test.failure.ignore \
                -DstaticAnalysis \
            """
            // step([$class: 'CheckStylePublisher', pattern: '**/target/checkstyle-result.xml', unstableTotalAll:'0'])
            // step([$class: 'PmdPublisher', pattern: '**/target/pmd.xml', unstableTotalAll:'0'])
            // step([$class: 'FindBugsPublisher', pattern: '**/findbugsXml.xml', unstableTotalAll:'0'])
            junit testResults: "**/${surefireTestReports}"
            // skip coverage report if unstable
            if (currentBuild.result == null) {
              step([ $class: 'JacocoPublisher' ])
            }
            notify.testResults("UNIT")
          }
        } catch (e) {
          currentBuild.result='FAILURE'
          notify.failed()
          throw e
        }
      }
    }
  }
}
