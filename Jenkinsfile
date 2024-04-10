@Library('piper-lib-os') _

node() {
    environment {
        GITHUB_APP_CREDENTIAL = credentials('github')
	configOptions = ''
    }
    stage('Prepare') {
        checkout scm
        setupCommonPipelineEnvironment script: this
    }
    stage('Initialize') {
        deleteDir()
        checkout scm
    }
	stage ('Pipeline'){
		script{
			def config = readYaml file: './.pipeline/configArtefacts.yml'
			for (def step in config.steps) {
				def configOptions = [
				cpiApiServiceKeyCredentialsId : step.cpiApiServiceKeyCredentialsId,
  				integrationFlowId : step.integrationFlowId,
				integrationFlowVersion : step.integrationFlowVersion,
				downloadPath : step.downloadPath
				]

				echo "Config Options: ${configOptions}"
				stage('IntegrationArtifactDownload Command') {
					integrationArtifactDownload (configOptions)
				}
				
				stage('Commit and Push to GitHub') {
					sh 'git config --global user.email "your-email@example.com"'
                    			sh 'git config --global user.name "Jenkins"'

					echo "Config Options in stage commit: ${configOptions}"
			
					integrationFlowId = configOptions.integrationFlowId
					packageId = step.packageId
					echo " directory integrationflowid ${integrationFlowId}  "
					// Clone the GitHub repository to a temporary directory
  					dir("IntegrationContent/${packageId}/${integrationFlowId}"){
						sh "mkdir -p /var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}"
						sh "unzip -o -q /var/lib/jenkins/workspace/IntegrationArtifactDownload/${integrationFlowId}/* -d /var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}"
						sh "cp -r	/var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}/* ."
						sh "git add ."
					}	
					
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'github' ,usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {  
						sh 'git diff-index --quiet HEAD || git commit -am ' + '\'' + 'commit files' + '\''
						sh('git push https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@' + 'github.com/Pan-Pac-Forest-Products/panpac-cpi-integrationArtefact-download.git' + ' HEAD:' + 'main')
					}
				}
			}
		}
	}
}
