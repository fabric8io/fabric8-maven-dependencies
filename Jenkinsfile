#!/usr/bin/groovy
@Library('github.com/rawlingsj/fabric8-pipeline-library@master')
def dummy
clientsNode {
  ws {
    checkout scm
    sh "git remote set-url origin git@github.com:fabric8io/fabric8-maven-dependencies.git"

    def pipeline = load 'release.groovy'

    pipeline.updateDependencies('fabric8-quickstarts')
  }
}
