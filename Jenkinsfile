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
    deleteDir()
    checkout scm
    dir("/home/ec2-user/relay42-infra") {
      git branch: 'master', credentialsId: 'github-ssh', url: 'git@github.com:muffat/relay42-infra.git'
    }
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
		app_version = "$commitId".substring(0,7)
		currentBuild.displayName = "#$BUILD_NUMBER -"+" $app_version"

    sh "./mvnw clean package spring-boot:repackage"
    sh "cp Dockerfile ./target/ && sudo docker build --build-arg JAR_FILE=target/*.jar -t hello-app:$app_version ."
    sh "sudo \$(aws ecr get-login --no-include-email --region ap-southeast-1)"
    sh "sudo docker tag hello-app:$app_version 824744317017.dkr.ecr.ap-southeast-1.amazonaws.com/hello-app:$app_version"
    sh "sudo docker push 824744317017.dkr.ecr.ap-southeast-1.amazonaws.com/hello-app:$app_version"
  }

  stage("tf plan") {
    task_desired_count = sh(returnStdout: true, script: 'aws ecs describe-services --cluster hello-app-abc --region ap-southeast-1 --services hello-app | grep desiredCount | grep -Eo \'[0-9]+\' | head -n1').toInteger()
    asg_desired_capacity = sh(returnStdout: true, script: 'aws ecs describe-clusters --clusters hello-app-abc --region ap-southeast-1 | grep registeredContainerInstancesCount | grep -Eo \'[0-9]+\'').toInteger()
    sh "cd ~/relay42-infra/hello-app/tf && \
    terraform get && \
    terraform init && \
    terraform plan \
    -var docker_image=824744317017.dkr.ecr.ap-southeast-1.amazonaws.com/hello-app:$app_version \
    -var task_desired_count=$task_desired_count \
    -var asg_desired_capacity=$asg_desired_capacity"
  }

  stage("tf apply") {
    def userInput = true
    def didTimeout = false
    try {
      timeout(time: 600, unit: 'SECONDS') {
        userInput = input(
          id: 'Proceed1', message: 'Continue?', parameters: [
            [
              $class: 'BooleanParameterDefinition',
              defaultValue: false,
              description: '',
              name: 'Please confirm you agree to deploy']
          ]
        )
      }
    } catch(err) {
      def user = err.getCauses()[0].getUser()
      if('SYSTEM' == user.toString()) {
        didTimeout = true
      } else {
        userInput = false
        echo "Aborted by: [${user}]"
      }
    }

    if (didTimeout) {
      echo "no input was received before timeout"
    } else if (userInput == true) {
      sh "cd ~/relay42-infra/hello-app/tf && \
      terraform get && \
      terraform init && \
      terraform apply -auto-approve \
      -var docker_image=824744317017.dkr.ecr.ap-southeast-1.amazonaws.com/hello-app:$app_version \
      -var task_desired_count=$task_desired_count \
      -var asg_desired_capacity=$asg_desired_capacity"
    } else {
      echo "this was not successful"
      currentBuild.result = 'FAILURE'
    }
  }
}