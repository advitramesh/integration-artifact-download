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
					// Clone the GitHub repository to a temporary directory
  					dir("IntegrationContent/${packageId}/${integrationFlowId}"){

                        		// Debug: Print the current working directory
        				sh 'pwd'
                        	       // Capture the output of 'pwd'
					def zipFolder = ''							
                        		zipFolder = sh(script: 'pwd', returnStdout: true).trim()

					// List the contents of the zipFolder
					echo "Listing contents of ${zipFolder}:"
					sh "ls -la ${zipFolder}"
					
					def destinationDir = "/var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}"	
					sh "mkdir -p ${destinationDir}"

					dir(zipFolder) {
    					// List all zip files in the directory
    					def zipFiles = sh(script: "ls *.zip", returnStdout: true).trim().split('\n')

    					// Iterate over each file and unzip
    					zipFiles.each { zipFileName ->
        				def zipFilePath = "${zipFolder}/${zipFileName}"
        				echo "Unzipping ${zipFilePath} to ${destinationDir}"
        				unzip zipFile: zipFilePath, dir: destinationDir
					sh "rm -f ${zipFilePath}"	
    						}
					}
						sh "cp -r	/var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}/* ."
						sh "git add ."
						// Commit the changes
            					sh 'git commit -m "Add latest iFlows for ${packageName}"'
					}	
					
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '613fd18c-2469-433c-bca6-22c48b4eb948' ,usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {  

						// Configure git to handle divergences
        					//sh 'git config pull.rebase false' // or 'true' if you prefer to rebase	

						 // Stash any local changes
        					//sh 'git stash'
						
						// Pull the latest changes from the remote repository
    						//sh ('git pull https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@github.com/advitramesh/cpi-dev.git main')

						// Re-apply the stashed changes
        					//sh 'git stash pop'
						
						sh 'git diff-index --quiet HEAD || git commit -am ' + '\'' + 'commit files' + '\''
						sh('git push https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@' + 'github.com/advitramesh/cpi-dev.git' + ' HEAD:' + 'main')
					}
				}
			}
		}
	}
}
