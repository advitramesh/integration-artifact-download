@Library('piper-lib-os') _

// Define the GitHub repository URL and branch as global variables
def repoUrl = 'https://github.com/advitramesh/cpi-dev.git'
def branchName = 'main'
def gitCredentialsId = '613fd18c-2469-433c-bca6-22c48b4eb948'

node() {
    environment {
        GITHUB_APP_CREDENTIAL = credentials(gitCredentialsId)
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

    stage('Pipeline') {
        script {
            def config = readYaml file: './.pipeline/configArtefacts.yml'
            for (def step in config.steps) {
                def configOptions = [
                    cpiApiServiceKeyCredentialsId: step.cpiApiServiceKeyCredentialsId,
                    integrationFlowId: step.integrationFlowId,
                    integrationFlowVersion: step.integrationFlowVersion,
                    downloadPath: step.downloadPath
                ]

                echo "Config Options: ${configOptions}"
                stage('IntegrationArtifactDownload Command') {
                    integrationArtifactDownload(configOptions)
                }

                def packageId = step.packageId 
                
                stage('Unzip iFlows') {
                    def zipFolder = configOptions.downloadPath
                    def destinationDir = "/var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}"
                    sh "mkdir -p ${destinationDir}"
                    dir(zipFolder) {
                        def zipFiles = sh(script: "ls *.zip", returnStdout: true).trim().split('\n')
                        zipFiles.each { zipFileName ->
                            def zipFilePath = "${zipFolder}/${zipFileName}"
                            echo "Unzipping ${zipFilePath} to ${destinationDir}"
                            unzip zipFile: zipFilePath, dir: destinationDir
                            sh "rm -f ${zipFilePath}"
                        }
                    }
                }

                stage('Copy iFlows to Git Repo') {
                    dir('gitRepo') {
                        sh "cp -r /var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}/* ."
                    }
                }

                stage('Stage the changes for commit') {
                    dir('gitRepo') {
                        sh 'git config --global user.email "advit.ramesh@accenture.com"'
                        sh 'git config --global user.name "advitramesh"'
                        sh "git add ."
                        sh 'git diff-index --quiet HEAD || git commit -am "Commit integration content updates"'


                        // Ensure you are on the correct branch and that it exists
                        sh "git fetch --all"
                        def hasMainBranch = sh(script: "git branch -r | grep 'origin/${branchName}'", returnStatus: true) == 0
                        if (hasMainBranch) {
                            sh "git checkout ${branchName}"
                        } else {
                            sh "git checkout -b ${branchName}"
                        }
                        

                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: gitCredentialsId, usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {
                            // Fetch and rebase (or merge) changes from remote
                            sh "git fetch origin ${branchName}"
                            sh "git rebase origin/${branchName}"
                            sh "git push https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@${repoUrl.replace('https://', '')} ${branchName}"
                        }
                    }
                }
            }
        }
    }
}
