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
    task_desired_count = sh(returnStdout: true, script: 'aws ecs describe-clusters --clusters hello-app --region ap-southeast-1 | grep runningTasksCount | grep -Eo \'[0-9]+\'').toInteger()
    asg_desired_capacity = sh(returnStdout: true, script: 'aws ecs describe-clusters --clusters hello-app --region ap-southeast-1 | grep registeredContainerInstancesCount | grep -Eo \'[0-9]+\'').toInteger()
    sh "cd ~/relay42-infra/hello-app/tf && \
    terraform get && \
    terraform init && \
    terraform plan \
    -var docker_image=824744317017.dkr.ecr.ap-southeast-1.amazonaws.com/hello-app:$app_version \
    -var task_desired_count=$task_desired_count \
    -var asg_desired_capacity=$asg_desired_capacity"
  }

  stage("tf apply") {
    sh "cd ~/relay42-infra/hello-app/tf && \
    terraform get && \
    terraform init && \
    terraform apply -auto-approve \
    -var docker_image=824744317017.dkr.ecr.ap-southeast-1.amazonaws.com/hello-app:$app_version \
    -var task_desired_count=$task_desired_count \
    -var asg_desired_capacity=$asg_desired_capacity"
  }
}