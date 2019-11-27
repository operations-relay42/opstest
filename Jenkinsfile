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
  stage("checkout") {
    checkout scm
  }

  stage("Build") {
    sh "./mvnw clean package spring-boot:repackage"
  }
}

