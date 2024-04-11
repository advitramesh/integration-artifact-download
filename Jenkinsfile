@Library('piper-lib-os') _

node() {
    // Environment variables
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

    stage('Pipeline') {
        script {
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

                stage('Unzip and Prepare for Commit') {
                    // Assuming the download path is the workspace root
                    def downloadDir = "${WORKSPACE}"
                    def artifactZipPath = "${downloadDir}/${step.integrationFlowId}.zip"

                    // Unzip the artifact
                    if (fileExists(artifactZipPath)) {
                        unzip zipFile: artifactZipPath, dir: downloadDir
                    } else {
                        echo "Artifact .zip not found in ${artifactZipPath}"
                    }

                    // Move the artifacts to the desired location
                    def localArtifactPath = "IntegrationContent/IntegrationArtefacts/${step.packageId}/${step.integrationFlowId}"
                    sh "mkdir -p ${localArtifactPath}"
                    sh "cp -r ${downloadDir}/* ${localArtifactPath}"

                    // Stage the changes
                    sh "git add ${localArtifactPath}"
                }

                stage('Commit and Push to GitHub') {
                    // Configure Git
                    sh 'git config --global user.email "advit.ramesh@accenture.com"'
                    sh 'git config --global user.name "advitramesh"'

                    // Commit and push
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: GITHUB_APP_CREDENTIAL, usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {
                        sh 'git diff-index --quiet HEAD || git commit -am "Add integration artifacts"'
                        sh "git push https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@github.com/advitramesh/cpi-dev.git HEAD:main"
                    }
                }
            }
        }
    }
}
