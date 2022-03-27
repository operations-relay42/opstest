@Library("shared-library") _

node {
   stage ('Checkout'){
        checkout scm
        sh 'git clean -fdx'
    }
    buildStage ('Build'){
        echo "building opstest"
		sh 'mvn -f opstest//pom.xml clean package'
        echo "building opstest"
        sh 'mvn -f opstest//pom.xml clean install'
    }
}

void buildStage(String context, Closure closure) {
    try {
        stage(context) {
            setBuildStatus(context, "In progress", "PENDING");
            closure();
            setBuildStatus(context, "Success", "SUCCESS");
        }
    } catch (Exception e) {
        multiBranchSendMail()
	setBuildStatus(context, "Failed", "FAILURE");
        throw e;
    }
}

void setBuildStatus(String context, String message, String state) {
    step([
        $class: "GitHubCommitStatusSetter",
        contextSource: [$class: "ManuallyEnteredCommitContextSource", context: context],
        reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/operations-relay42/opstest.git"],
        errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
        statusResultSource: [$class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]]]
    ]);
}
