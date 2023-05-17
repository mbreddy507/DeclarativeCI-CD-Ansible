def mvn
def server = Artifactory.server 'artifactory'
def rtMaven = Artifactory.newMavenBuild()
def buildInfo
def DockerTag() {
	def tag = sh script: 'git rev-parse HEAD', returnStdout:true
	return tag
	}
pipeline {
  agent { label 'master' }
    tools {
      maven 'Maven'
      jdk 'JAVA_HOME'
    }
}