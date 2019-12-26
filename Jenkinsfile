properties(
 [
  parameters(
   [
    string(
     defaultValue: '',
     description: 'commit version to build',
     name: 'version',
     trim: true),
        choice(
          choices: ["ap-southeast-1", "us-east-1"],
          description: 'select region',
          name: 'region')
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
 agent {
  node {
   label 'ec2-slave'
  }
 }

 environment {
  AWS_ACCOUNT_NUMBER = "824744317017"
 }

 stages {
   /*
   stage("Prepare") {
     steps {
     script {
       
      regions = []
     if (params.singapore == true) {
      regions.add('ap-southeast-1')
     }
          if (params.north_virginia == true) {
      regions.add('us-east-1')
     }
     
     }
     }
   }*/

  stage("Checkout") {
   steps {
    script {
     deleteDir()
     checkout scm
     sh "rm -rf /home/ec2-user/relay42-infra"
     dir("/home/ec2-user/relay42-infra") {
      git branch: 'develop', credentialsId: 'github-ssh', url: 'git@github.com:muffat/relay42-infra.git'
     }
    }
   }
  }

  stage("Build") {
   steps {
    script {
     if (params.version == "") {
      commitId = sh(returnStdout: true, script: 'git rev-parse HEAD')
     } else {
      commitId = params.version
     }
     commitId = commitId.trim()
     app_version = "$commitId".substring(0, 7)
     sh "sudo docker build -t hello-app:$app_version ."
    }
   }
  }

  stage("Publish") {
   steps {
    script {
     currentBuild.displayName = "#$BUILD_NUMBER -" + " $app_version"
     //regions.each { item ->
     //def region_name = "${item}"
     
     sh "sudo \$(aws ecr get-login --no-include-email --region us-east-1)"
     sh "sudo docker tag hello-app:$app_version ${env.AWS_ACCOUNT_NUMBER}.dkr.ecr.us-east-1.amazonaws.com/hello-app:$app_version"
     sh "sudo docker push ${env.AWS_ACCOUNT_NUMBER}.dkr.ecr.us-east-1.amazonaws.com/hello-app:$app_version"
   }
   }
  }

  stage("Deploy to DEV") {
   steps {
    script {

     //task_desired_count = sh(returnStdout: true, script: "aws ecs describe-services --cluster hello-app-dev --region ap-southeast-1 --services hello-app | grep desiredCount | grep -Eo '[0-9]+' | head -n1").toInteger()
     //asg_desired_capacity = sh(returnStdout: true, script: "aws ecs describe-clusters --clusters hello-app-dev --region ap-southeast-1 | grep registeredContainerInstancesCount | grep -Eo '[0-9]+'").toInteger()
     terraform_plan(app_version, params.region, "dev")
     terraform_apply(app_version, params.region, "dev")

    }
   }
   //}
  }
 }
}

def terraform_plan(app_version, region, env) {
  sh "cd ~/relay42-infra/hello-app/tf && \
		terraform get && \
		terraform init && \
		terraform workspace select ${region}-${env} && \
		terraform plan -var docker_image=824744317017.dkr.ecr.us-east-1.amazonaws.com/hello-app:${app_version} \
		-var environment=${env} \
		-var-file=regions/${region}.tfvars"
}

def terraform_apply(app_version, region, env) {
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
            name: 'Please confirm you agree to deploy'
          ]
        ]
      )
    }
  } catch (err) {
    def user = err.getCauses()[0].getUser()
    if ('SYSTEM' == user.toString()) {
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
			terraform workspace select ${region}-${env} && \
			terraform apply \
			-var docker_image=824744317017.dkr.ecr.us-east-1.amazonaws.com/hello-app:${app_version} \
		-var environment=${env} \
		-var-file=regions/${region}.tfvars \
			-auto-approve"
  } else {
    echo "this was not successful"
    currentBuild.result = 'FAILURE'
  }
}