import groovy.json.JsonSlurper
import groovy.json.*
import java.io.OutputStreamWriter
import java.lang.String
import groovy.transform.Field
import jenkins.model.*
import java.io.FileReader
import java.util.Iterator
import java.util.Map
import groovy.io.FileType
import groovy.json.JsonSlurperClassic
import java.net.URLEncoder
import java.util.zip.ZipOutputStream
import java.util.zip.ZipEntry
import java.nio.file.Files
import java.nio.file.Paths


def ZIP_NODE
def ZIP_WORKFLOW

 def jsonResponse
 String fileuploadUrl
 String postuploadfileuploadUrl
 def list = []

 String taskID
 String zipfilepath
 String zip_workflowfilepath
 String includefilenames
 String includedfile=''
 String filedata
 def jsonIncludefilepath 
 def postdeploymentfilepath
 def statuscode
 def objectstatus
 def objectsecondstatus
 String statusDetail1msg ="";
 def status_final

pipeline {
	agent any
	/*stages {
		stage("build") {
			steps {
				echo 'building the application...'
			}
		}
		stage("test") {
			steps {
				echo 'testing the application...'
			}
		}
		stage("deploy") {
			steps {
				echo 'deployin the application...'
			}
		}
	}*/
	stages {
		stage('Initialization...'){
            steps {
                script {
                    echo "====Initialization start..====="
                    ZIP_NODE="${env.BRANCH_NAME}.zip"
                    ZIP_WORKFLOW ="postdeployment.zip"
                    
                    jsonIncludefilepath = "${env.WORKSPACE}/deployment-artifacts/includedfile.json"
                    postdeploymentfilepath = "${env.WORKSPACE}/postdeployment/postdeploymentconfig.json"
                         
                    zipfilepath= "${env.WORKSPACE}/${ZIP_NODE}"
                    
                    zip_workflowfilepath ="${env.WORKSPACE}/${ZIP_WORKFLOW}"
                    
                    println("ZIP_NODE:"+ZIP_NODE)
                    println("ZIP_WORKFLOW:"+ZIP_WORKFLOW)
                    println("jsonIncludefilepath:"+jsonIncludefilepath)
                    println("postdeploymentfilepath:"+postdeploymentfilepath)
                    println("zipfilepath:"+zipfilepath)
                    println("zip_workflowfilepath:"+zip_workflowfilepath)
                }
            }
        }
            stage('prepare-package') {
            steps {
                script {
                    // Read filenames from includedfile.json
                    def includedFileContent = readFile("${jsonIncludefilepath}")
                    def jsonSlurper = new groovy.json.JsonSlurper()
                    def includedFiles = jsonSlurper.parseText(includedFileContent)

                    // Create a list to store the filenames
                    def filenames = []

                    // Extract filenames from includedFiles
                    includedFiles.each { file ->
                        filenames.add(file.filename)
                    }

                    // Join filenames using space as separator
                    def filenamesString = filenames.join(' ')

                    // Create the zip file using Windows CMD
                    bat "cd ${WORKSPACE} && powershell -Command \"Compress-Archive -Path ${filenamesString} -DestinationPath ${zipfilepath}\""
                    def zipResult = bat(returnStdout: true, script: zipCommand)

                    // Check if zip file is created
                    if (!fileExists(ZIP_NODE)) {
                        error("Failed to create the zip file.")
                    }
                }
            }
        }

	}
}
