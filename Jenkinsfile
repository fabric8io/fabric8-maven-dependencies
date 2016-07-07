#!/usr/bin/groovy
import groovy.xml.DOMBuilder
import groovy.xml.XmlUtil
import groovy.xml.dom.DOMCategory
import org.w3c.dom.Element

node {
  ws {
    checkout scm
    sh "git remote set-url origin git@github.com:fabric8io/fabric8-maven-dependencies.git"

    def organisation = 'fabric8-quickstarts'
    def pomLocation = 'pom.xml'

    def flow = new io.fabric8.Fabric8Commands()

    def replaceVersions = loadPomPropertyVersions(pomLocation)

    println "About to try replace versions: ${replaceVersions}"

    if (replaceVersions.size() > 0) {
      println "Now updating all projects within organisation: ${organisation}"

      repoApi = new URL("https://api.github.com/orgs/${organisation}/repos")
      repos = new groovy.json.JsonSlurper().parse(repoApi.newReader())

      for (repoData in repos) {
        def repo = repoData.name
        println "Processing repo: ${repo}"

        // lets check if the repo has a pom.xml
        pomUrl = new URL("https://raw.githubusercontent.com/${organisation}/${repo}/master/pom.xml")
        def hasPom = false
        try {
          hasPom = !pomUrl.text.isEmpty()
        } catch (e) {
          // ignore
        }

        if (hasPom) {
          def project = "${organisation}/${repo}"

          stage "Updating ${project}"
          sh "rm -rf ${repo}"
          sh "git clone https://github.com/${project}.git"
          sh "cd ${repo} && git remote set-url origin git@github.com:${project}.git"

          def uid = UUID.randomUUID().toString()
          sh "cd ${repo} && git checkout -b versionUpdate${uid}"

          def xml = readFile file: "${repo}/${pomLocation}"
          //sh "cat ${repo}/${pomLocation}"

          def changed = false
          for (entry in replaceVersions) {
            println("replacing property ${entry.key} with value: ${entry.value}")
            println("replacing property ${entry.key} with value: ${entry.value}")
            def pom = updateVersion(xml, entry.key, entry.value)
            if (pom != null) {
              xml = pom
              changed = true
            }
          }
          if (changed) {
            writeFile file: "${repo}/${pomLocation}", text: xml
            echo "updated file ${repo}/${pomLocation}"

            //sh "cat ${repo}/${pomLocation}"

            kubernetes.pod('buildpod').withImage('fabric8/maven-builder:latest')
                    .withPrivileged(true)
                    .withSecret('jenkins-git-ssh', '/root/.ssh-git')
                    .withSecret('jenkins-ssh-config', '/root/.ssh')
                    .inside {

              sh 'chmod 600 /root/.ssh-git/ssh-key'
              sh 'chmod 600 /root/.ssh-git/ssh-key.pub'
              sh 'chmod 700 /root/.ssh-git'

              sh "git config --global user.email fabric8-admin@googlegroups.com"
              sh "git config --global user.name fabric8-release"

              def githubToken = flow.getGitHubToken()
              def message = "\"Update pom property versions\""
              sh "cd ${repo} && git add ${pomLocation}"
              sh "cd ${repo} && git commit -m ${message}"
              sh "cd ${repo} && git push origin versionUpdate${uid}"
              retry(5) {
                sh "export GITHUB_TOKEN=${githubToken} && cd ${repo} && hub pull-request -m ${message} > pr.txt"
              }
            }
            pr = readFile("${repo}/pr.txt")
            split = pr.split('\\/')
            def prId = split[6].trim()
            echo "received Pull Request Id: ${prId}"
            flow.addMergeCommentToPullRequest(prId, project)
          }
        }
      }
    }
  }
}

@NonCPS
def loadPomPropertyVersions(fileName) {
  echo "Finding property versions in file ${fileName}"

  def answer = [:]
  def localPomXml = readFile file: fileName
  localPomXml.take(localPomXml.indexOf('<project'))
  def xmlDom = DOMBuilder.newInstance().parseText(localPomXml)
  def propertiesList = xmlDom.getElementsByTagName("properties")
  if (propertiesList.length == 0) {
    echo "No <properties> element found in pom.xml!"
  } else {
    def propertiesElement = propertiesList.item(0)
    for (node in propertiesElement.childNodes) {
      if (node instanceof Element) {
        answer[node.nodeName] = node.textContent
      }
    }
  }
  echo "Have loaded replaceVersions ${answer}"
  return answer
}

@NonCPS
def updateVersion(xml, elementName, newVersion) {
  def index = xml.indexOf('<project')
  def header = xml.take(index)
  def xmlDom = DOMBuilder.newInstance().parseText(xml)
  def root = xmlDom.documentElement
  use(DOMCategory) {
    def versions = xmlDom.getElementsByTagName(elementName)
    if (versions.length == 0) {
      echo "No element found called ${elementName}"
      return null
    } else {
      def version = versions.item(0)
      echo "version ${elementName} = ${version.textContent}"
      if (newVersion != version.textContent) {
        version.textContent = newVersion

        def newXml = XmlUtil.serialize(root)
        // need to fix this, we get errors above then next time round if this is left in
        return header + newXml.minus('<?xml version="1.0" encoding="UTF-8"?>')
      } else {
        return null
      }
    }
  }
}
