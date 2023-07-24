def mvn
//def server = Artifactory.server 'artifactory'
//def rtMaven = Artifactory.newMavenBuild()
//def buildInfo
def DockerTag() {
	def tag = sh script: 'git rev-parse HEAD', returnStdout:true
	return tag
	}
pipeline 
{
  agent any
    tools {
      maven 'Maven'
      jdk 'JAVA_HOME'
    }
  options { 
    timestamps () 
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '5')	
// numToKeepStr - Max # of builds to keep
// daysToKeepStr - Days to keep builds
// artifactDaysToKeepStr - Days to keep artifacts
// artifactNumToKeepStr - Max # of builds to keep with artifacts	  
}	
  environment {
    SONAR_HOME = "${tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'}"
    DOCKER_TAG = DockerTag()	  
  }  
  stages {
	  stage ('Maven Build') {
      steps {
        script {
          mvn= tool (name: 'Maven', type: 'maven') + '/bin/mvn'
        }
        sh "${mvn} clean package"
      }
    }
    stage('Artifactory_Configuration') {
      steps {
        sh 'echo passed'
        //script {
	//	  rtMaven.tool = 'Maven'
	//	  rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
	//	  buildInfo = Artifactory.newBuildInfo()
	//	  rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot', server: server
         // buildInfo.env.capture = true
        //}			                      
      }
    }
    stage('Execute_Maven') {
	  steps {
      sh 'echo pass'
	  //  script {
	//	  rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
        //}			                      
      }
    }	
    stage('War rename') {
	  steps {
		  	sh 'mv target/*.war target/helloworld.war'
        }			                      
      }
    stage('SonarQube_Analysis') {
      steps {
	    script {
          scannerHome = tool 'sonar-scanner'
        }
        withSonarQubeEnv('sonar') {
      	  sh """${scannerHome}/bin/sonar-scanner"""
        }
      }	
    }	
	stage('Quality_Gate') {
	  steps {
	    timeout(time: 3, unit: 'MINUTES') {
		  waitForQualityGate abortPipeline: true
        }
      }
    }
  stage('Build Docker Image'){
    steps{
      sh 'docker build -t mbreddy507/ansibledeploy:${DOCKER_TAG} .'
    }
  }	  	 
  stage('Docker Container'){
    steps{
      withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'docker_pass', usernameVariable: 'docker_user')]) {
	  sh 'docker login -u ${docker_user} -p ${docker_pass}'
      	  sh 'docker push mbreddy507/ansibledeploy:${DOCKER_TAG}'
	  }
    }
 }
    stage('Ansible Playbook')
    {
      steps {
        sh 'echo Pass'
       //ansiblePlaybook credentialsId: 'ans-server', inventory: 'inventory', playbook: 'ansibleplay.yml', tags: 'stop_container,delete_container'
       // Above command used to run the playbook with specified tags mentioned in the tags section.	
	 //ansiblePlaybook credentialsId: 'ans-server', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory', playbook: 'ansibleplay.yml'     
     //   }
      }	  	  
  }

stage('Update Deployment File') 
{
   environment {
            GIT_REPO_NAME = "DeclarativeCI-CD-Ansible"
            GIT_USER_NAME = "mbreddy507"
            GIT_PASSWORD="Anushka@19891990"
            GIT_REPO_URL="https://github.com/mbreddy507/DeclarativeCI-CD-Ansible.git"
        }
  steps {
    withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
sh '''
                    git config user.email "mbreddy507@gmail.com"
                    git config user.name "mbreddy507"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/ImageTag/${BUILD_NUMBER}/g" deployment.yml
                    git add deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@${GIT_REPO_URL} HEAD:task-12"
                '''
  } 
  }
}
}
}



