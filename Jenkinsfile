@Library('piper-lib-os') _

// Define the GitHub repository URL and branch as global variables

def repoUrl = 'https://github.com/advitramesh/cpi-dev.git'
def branchName = 'main'
def gitCredentialsId = '613fd18c-2469-433c-bca6-22c48b4eb948'
node() {
    environment {
        GITHUB_APP_CREDENTIAL = credentials('613fd18c-2469-433c-bca6-22c48b4eb948')
	configOptions = ''

	    
    }
    stage('Prepare') {
        checkout scm
        setupCommonPipelineEnvironment script: this
    }
    // Clone the GitHub Repository
    stage('Clone GitHub Repo') {
        dir('gitRepo') {
            withCredentials([usernamePassword(credentialsId: gitCredentialsId, passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                // Use the credentials to clone
                sh "git clone -b ${branchName} https://${GIT_USERNAME}:${GIT_PASSWORD}@${repoUrl.replace('https://', '')} ."
            }
        }
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
					

					echo "Config Options in stage commit: ${configOptions}"
			
					integrationFlowId = configOptions.integrationFlowId
					packageId = step.packageId
					echo " directory integrationflowid ${integrationFlowId}  "
					// Clone the GitHub repository to a temporary directory
  					dir("IntegrationContent/${packageId}/${integrationFlowId}"){

                        		def zipFolder = configOptions.downloadPath							
                        		
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


					sh "cp -r /var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}/* ."

					// Copy the downloaded iFlows to the cloned repository
					stage('Copy iFlows to Git Repo') {
                    				dir('gitRepo') {
            					// Copy iFlows to the 'gitRepo/' directory
            							sh "cp -r /var/lib/jenkins/workspace/IntegrationContent/ZESPRICanopySAPBW/GET_CRM_Data/* ."
        					}
                			}	

					// Stage the changes for commit
 					 stage('Stage the changes for commit') {
        				dir('gitRepo') {
						// Configure Git user
						sh 'git config --global user.email "advit.ramesh@accenture.com"'
                    				sh 'git config --global user.name "advitramesh"'
					 
						// Add and commit changes		
						sh "git add ."

						sh 'git diff-index --quiet HEAD || git commit -am "Commit integration content updates"'

						// Ensure you are on the correct branch
            					sh "git checkout ${branchName}"
				
						// Push changes		
						withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '613fd18c-2469-433c-bca6-22c48b4eb948' ,usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {  
						
						//sh 'git diff-index --quiet HEAD || git commit -am ' + '\'' + 'commit files' + '\''
						sh 'git push https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@${repoUrl} ${branchName}'
						
					
						}
					}
				}
		}
	}
}
