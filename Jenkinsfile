properties(
  [
    parameters(
      [
			  string(
					defaultValue: '',
					description: 'commit version to build',
					name: 'version',
					trim: true)
      ]
    ),
		disableConcurrentBuilds(),
		pipelineTriggers(
		  [
				githubPush()
			]
		)
  ]
)

node("ec2-slave") {
  stage("Checkout") {
    checkout scm
  }

  stage("Build") {
    sh "rm -rf ./target"
    sh "./mvnw clean package spring-boot:repackage"
  }

  stage("Publish") {
		if (params.version == "") {
			commitId = sh(returnStdout: true, script: 'git rev-parse HEAD')
		} else {
			commitId = params.version
		}
    commitId = commitId.trim()
		def app_version = "$commitId".substring(0,7)
		currentBuild.displayName = "#$BUILD_NUMBER -"+" $app_version"

    commit_version = "$version".substring(0,7)
    sh "./mvnw clean package spring-boot:repackage"
    sh '''
    cp Dockerfile ./target/ \
    && sudo docker build -t hello-app:$app_version .
    '''
  }
}
/*
pipeline {
  agent none
  stages {
    stage('Test') {
        agent { label 'ecs-slave'}
        steps {
            sh 'docker --version'
        }
    }
  }
}
*/

