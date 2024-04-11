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
                    cpiApiServiceKeyCredentialsId: step.cpiApiServiceKeyCredentialsId,
                    integrationFlowId: step.integrationFlowId,
                    integrationFlowVersion: step.integrationFlowVersion,
                    downloadPath: step.downloadPath
                ]

                echo "Config Options: ${configOptions}"
                stage('IntegrationArtifactDownload Command') {
                    integrationArtifactDownload(configOptions)
                }
                
                stage('Commit and Push to GitHub') {
                    sh 'git config --global user.email "advit.ramesh@accenture.com"'
                    sh 'git config --global user.name "advitramesh"'

                    echo "Config Options in stage commit: ${configOptions}"

                    integrationFlowId = configOptions.integrationFlowId
                    packageId = step.packageId

                    def downloadDir = "/var/lib/jenkins/workspace/SAPCPIArtifactDownload/${packageId}/${integrationFlowId}"
                    def extractionDir = "/var/lib/jenkins/workspace/IntegrationContent/${packageId}/${integrationFlowId}"

                    // Create directory for integration flow content
                    sh "mkdir -p ${extractionDir}"

                    // Check if the ZIP file exists and log the contents of the download directory
                    sh "ls -la ${downloadDir} || echo 'The specified directory does not exist or cannot be accessed.'"

                    // Attempt to unzip the artifact
                    try {
                        // Use the Pipeline Utility Steps Plugin to unzip the file
                        echo "Unzipping the artifact ZIP file"
                        unzip zipFile: "${downloadDir}/*.zip", dir: extractionDir
                    } catch (Exception e) {
                        echo "Unzip step failed with exception: ${e.getMessage()}"
                    }

                    // Change into the extraction directory and add all unzipped files to the Git repo
                    dir(extractionDir) {
                        sh "git add ."

                        // Commit and push if there are changes
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'GITHUB_APP_CREDENTIAL', usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {
                            sh 'git diff-index --quiet HEAD || git commit -am "Integration Artifacts update from CI/CD pipeline"'
                            sh "git push https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@github.com/advitramesh/cpi-dev.git HEAD:${env.GITBranch}"
                        }
                    }
                }
            }
        }
    }
}
