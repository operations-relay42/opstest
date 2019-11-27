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
    checkout scm
  }

  stage("Build") {
    sh "rm -rf ./target"
    sh "./mvnw clean package spring-boot:repackage"
  }

  stage("Publish") {
    sh "./mvnw clean package spring-boot:repackage"
  }
}

