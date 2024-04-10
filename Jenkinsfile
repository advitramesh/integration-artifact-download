@Library('piper-lib-os') _

node() {
    environment {
        GITHUB_APP_CREDENTIAL = credentials('613fd18c-2469-433c-bca6-22c48b4eb948')
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
					sh 'git config --global user.email "advit.ramesh@accenture.com"'
                    			sh 'git config --global user.name "advitramesh"'

					echo "Config Options in stage commit: ${configOptions}"
			
					integrationFlowId = configOptions.integrationFlowId
					packageId = step.packageId
					echo " directory integrationflowid ${integrationFlowId}  "

					// Debug: Check if unzip is available

					sh 'which unzip || echo "unzip command not found"'

					
					// Clone the GitHub repository to a temporary directory
  					dir("IntegrationContent/${packageId}/${integrationFlowId}"){
						// Debug: Print the current working directory
        					sh 'pwd'
						// Debug: List the contents of the download directory
        					sh "ls -la /var/lib/jenkins/workspace/SAPCPIArtifactDownload/IntegrationContent/${packageId}/${integrationFlowId}"
						// Create directory for integration flow content
						sh "mkdir -p /var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}"
						// Attempt to unzip
        					// Debug: Print the command being run
        					echo "Running unzip command:"
						
						sh "unzip -o -q /var/lib/jenkins/workspace/SAPCPIArtifactDownload/IntegrationContent/${packageId}/${integrationFlowId}/* -d /var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}"
						// Copy contents to current directory and add to Git
						sh "cp -r	/var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}/* ."
						sh "git add ."
					}	
					
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '613fd18c-2469-433c-bca6-22c48b4eb948' ,usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {  
						sh 'git diff-index --quiet HEAD || git commit -am ' + '\'' + 'commit files' + '\''
						sh('git push https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@' + 'github.com/advitramesh/cpi-dev/tree/main/IntegrationContent/IntegrationArtefacts' + ' HEAD:' + env.GITBranch)
					}
				}
			}
		}
	}
}
