import groovy.json.JsonSlurper

def ZIP_NODE
def ZIP_WORKFLOW
def jsonIncludefilepath
def zipfilepath
def zip_workflowfilepath

pipeline {
    agent any

    stages {
        stage('Initialization...') {
            steps {
                script {
                    ZIP_NODE = "${env.BRANCH_NAME}.zip"
                    ZIP_WORKFLOW = "postdeployment.zip"
                    jsonIncludefilepath = "${env.WORKSPACE}/deployment-artifacts/includedfile.json"
                    zipfilepath = "${env.WORKSPACE}/${ZIP_NODE}"
                    zip_workflowfilepath = "${env.WORKSPACE}/${ZIP_WORKFLOW}"

                    println("ZIP_NODE: " + ZIP_NODE)
                    println("ZIP_WORKFLOW: " + ZIP_WORKFLOW)
                    println("jsonIncludefilepath: " + jsonIncludefilepath)
                    println("zipfilepath: " + zipfilepath)
                    println("zip_workflowfilepath: " + zip_workflowfilepath)
                }
            }
        }
        stage('prepare-package') {
            steps {
                script {
                    def includedFileContent = readFile(jsonIncludefilepath)
                    def includedFiles = new JsonSlurper().parseText(includedFileContent)
                    def filenames = includedFiles.collect { it.filename }
                    def filenamesString = filenames.join(' ')

                    bat "powershell.exe -Command \"Compress-Archive -Path ${filenamesString} -DestinationPath ${zipfilepath}\""
                    if (!fileExists(ZIP_NODE)) {
                        error("Failed to create the zip file.")
                    }
                }
            }
        }
    }
}
