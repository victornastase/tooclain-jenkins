@Library('pfPipeline@tags/4') _

pipeline {

	agent {
		label 'java'
	}

	options {
		gitLabConnection('gitlab-dogen.group.echonet')
		disableConcurrentBuilds()
		buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5'))
		timeout(time: 1, unit: 'HOURS')
        // adds support for standard ANSI escape sequences, including color, to Console Output.
        ansiColor('xterm')
	}

	triggers {
		gitlab(triggerOnPush: true, branchFilterType: 'All')
	}

	parameters {
	    booleanParam(name: "DEPLOY_BRANCH", description: "Check to deploy this branch into DEV", defaultValue: false)
	    booleanParam(name: "QUALITY", description: "To trigger quality gate checks", defaultValue: true)
		booleanParam(name: "RELEASE", description: 'Build a release from current commit.', defaultValue: false)
	}

    stages {
        stage('build') {
            when {
                expression {
                    !params.RELEASE
                }
            }
            steps {
                pfMvnBuild(options: '-U')
            }
        }

		stage('deploy') {
            when {
                expression {
                    params.DEPLOY_BRANCH
                }
            }
            steps {
                pfWlb(
                    host: '<machine_hostname>',
                    name: '<app_name>',
                    ihsPort: '<app_port>',
                    https: 'true',
                    codeAP: '<BNP_APPLICATION_CODE>',
                    code5car: 'dili0',
                    libelle: '<wlb_folder>'
                )
            }
        }

        stage('release') {
            when {
                expression {
                    params.RELEASE
                }
            }
            steps {
                pfMvnRelease()
            }
        }

		stage('quality gate') {
            when {
                expression {
                    params.QUALITY
                }
            }
            steps {
                pfIQServerAnalysis (
                    iqApplication: readMavenPom().getArtifactId(),
                    iqStage: 'build'
                )
            }
        }
    }

}
