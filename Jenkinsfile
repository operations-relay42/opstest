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

node {
  stage("Checkout") {
        agent {
            ecs {
                inheritFrom 'java-ecs-slave'
            }
        }
  }
/*
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
  */
}

