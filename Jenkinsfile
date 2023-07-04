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
                    /*try{
                        def inputeIncludeFile = new File("${jsonIncludefilepath}")
                        def InputIncludeJSON = new JsonSlurper().parse(inputeIncludeFile)
                        includefilenames = InputIncludeJSON.filename
                        String[] arrOfIncludedfiles = includefilenames.split(","); 
                        for(int i=0; i< arrOfIncludedfiles.length; i++) {
                            def includedfileoutput  = "--include=*${arrOfIncludedfiles[i]}-*"
                            includedfile = includedfile + includedfileoutput + " "
                        }
                        println("includedfile variable done:"+includedfile)
                    } catch(Exception e) {
                    }*/
                    /*def inputeIncludeFile = new File("${jsonIncludefilepath}")
                        def InputIncludeJSON = new JsonSlurper().parse(inputeIncludeFile)
                        includefilenames = InputIncludeJSON.filename
                        String[] arrOfIncludedfiles = includefilenames.split(","); 
                        // Convert to serializable Map
                        def serializableIncludedFiles = arrOfIncludedfiles.collectEntries { filename ->
                            [(filename): true]
                        }
                    
                    if(includefilenames != null && !includefilenames.isEmpty()){
                        println("Running command to zip the file")
                        bat "echo ${ZIP_NODE} && echo 'remove alraedy existing zip files' && del *.zip && powershell Compress-Archive -Path * ${serializableIncludedFiles} -DestinationPath ${ZIP_NODE}"

                    } else {
                        println("includefilenames is")
                        bat "echo ${ZIP_NODE} && echo 'remove alraedy existing zip files' && del *.zip && powershell Compress-Archive -Path * -DestinationPath ${ZIP_NODE} -Exclude deployment-artifacts/, postdeployment/"
                    }*/
                    // Read the JSON file
                    def includedFileContent = readFile("${jsonIncludefilepath}")

                    // Parse the JSON content
                    def jsonSlurper = new JsonSlurper()
                    def json = jsonSlurper.parseText()includedFileContent

                    // Extract the filenames from the JSON data
                    def filenames = json.collect { it.filename }

                    // Define the output zip file path
                    def zipFilePath = "${ZIP_NODE}"

                    // Remove the existing zip file if it exists
                    def existingZipFile = new File(zipFilePath)
                    if (existingZipFile.exists()) {
                        existingZipFile.delete()
                        echo "Existing zip file deleted."
                    }

                    // Create a ZipOutputStream
                    def zipOutputStream = new ZipOutputStream(new FileOutputStream(zipFilePath))


                    // Iterate over the filenames and add each file to the zip
                    filenames.each { filename ->
                        def file = new File(filename)
                        if (file.exists()) {
                            def zipEntry = new ZipEntry(file.name)
                            zipOutputStream.putNextEntry(zipEntry)
                            zipOutputStream.write(Files.readAllBytes(Paths.get(file.toURI())))
                            zipOutputStream.closeEntry()
                        } else {
                            error "File not found: ${filename}"
                        }
                    }

                    // Close the ZipOutputStream
                    zipOutputStream.close()

                    echo "Files zipped successfully."
                } 
            }
        }

	}
}
