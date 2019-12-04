
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

node("ecs-slave") {
  stage("Checkout") {
    checkout scm
    sh 'pwd && ls -l'
  }

  stage("Build") {
    sh "rm -rf ./target"
    sh "./mvnw clean package spring-boot:repackage"
  }

  stage("Publish") {
    sh "./mvnw clean package spring-boot:repackage"
    sh '''
    cp Dockerfile ./target/ \
    && sudo docker build -t a .
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

