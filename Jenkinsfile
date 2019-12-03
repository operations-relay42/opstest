
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

pipeline {
  agent none

  stage("Checkout") {
        agent { label 'ecs-slave'}
        steps {
            checkout scm
            sh 'docker --version'
        }
  }

  stage("Build") {
    sh "rm -rf ./target"
    sh "./mvnw clean package spring-boot:repackage"
  }
  stage("Publish") {
    sh "./mvnw clean package spring-boot:repackage"
    sh '''
    cp Dockerfile ./target/ \
    && sudo docker build 
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

