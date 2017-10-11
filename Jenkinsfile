#!/usr/bin/env groovy

// Import pipeline library for utility methods & classes:
// ansicolor(), Notifier, PullRequests, Strings
@Field
public static final String PIPELINE_LIBRARY_BRANCH = 'ZNTA-2234-codecov'

@Library('github.com/zanata/zanata-pipeline-library@ZNTA-2234-codecov')
import org.zanata.jenkins.Notifier
import org.zanata.jenkins.PullRequests
import org.zanata.jenkins.Reporting
import static org.zanata.jenkins.StackTraces.getStackTrace

import groovy.transform.Field

PullRequests.ensureJobDescription(env, manager, steps)

@Field
def notify
// initialiser must be run separately (bindings not available during compilation phase)
notify = new Notifier(env, steps, currentBuild,
    'https://github.com/zanata/openprops.git',
    'Jenkinsfile', PIPELINE_LIBRARY_BRANCH,
)

@Field
def reporting
reporting = new Reporting(env, steps, 'https://github.com/zanata/openprops.git')

/* Only keep the 10 most recent builds. */
def projectProperties = [
  [
    $class: 'GithubProjectProperty',
    projectUrlStr: 'https://github.com/zanata/openprops'
  ],
  [
    $class: 'BuildDiscarderProperty',
    strategy: [$class: 'LogRotator', numToKeepStr: '10']
  ],
]
properties(projectProperties)

def surefireTestReports='target/surefire-reports/TEST-*.xml'

timestamps {
  node {
    echo "running on node ${env.NODE_NAME}"
    ansicolor {
      // ensure the build can handle at-signs in paths:
      dir("@") {
        try {
          stage('Checkout') {
            notify.started()
            checkout scm
          }

          stage('Check build tools') {
            // Note: if next stage happens on another node, mvnw might have to download again
            sh "./mvnw --version"
          }

          stage('Build') {
            notify.startBuilding()
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

            // send test coverage data to codecov.io
            reporting.codecov()

            // skip coverage report if unstable
            if (currentBuild.result == null) {
              step([ $class: 'JacocoPublisher' ])
            }
            notify.testResults('UNIT', currentBuild.result)
          }
        } catch (e) {
          currentBuild.result='FAILURE'
          notify.failed()
          throw e
        }
      }
    }
    notify.finish()
    currentBuild.result = (currentBuild.result) ?: 'SUCCESS'
  }
}
