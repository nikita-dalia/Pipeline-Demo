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
		stage("Initialization"){
			steps{
				script{
				
					echo "Initialization starting..."
					ZIP_NODE="${env.JOB_NAME}.zip"
					ZIP_WORKFLOW="postdeployment.zip"

					jsonIncludefilepath = "${env.WORKSPACE}/deployment-artifacts/includedfile.json"
					postdeploymentfilepath = "${env.WORKSPACE}/postdeployment/postdeploymentconfig.json"
							
					zipfilepath= "${env.WORKSPACE}/${ZIP_NODE}"
					
					zip_workflowfilepath ="${env.WORKSPACE}/${ZIP_WORKFLOW}"
					
					println(zip_workflowfilepath)
				}

			}
		}
		stage("Prepare Package"){
			steps{
				echo "Preparing Package Stage..."
				script {
                     
                     try{
                         
                         def inputeIncludeFile = new File("${jsonIncludefilepath}")
                         def InputIncludeJSON = new JsonSlurper().parse(inputeIncludeFile)
                         includefilenames = InputIncludeJSON.filename
                         String[] arrOfIncludedfiles = includefilenames.split(","); 
                         for(int i=0; i< arrOfIncludedfiles.length; i++)
                           {
                                def includedfileoutput  = "--include"+"=*"+arrOfIncludedfiles[i]-'['-']'-' '+"*";
                    
                                   includedfile = includedfile+includedfileoutput+" "
                                 //  println("data["+i+"] : "+arrOfIncludedfiles[i])
                          }
                          
                        }catch(Exception e) {
                       }
                
                      //  println(includefilenames)
                      if(includefilenames != null && !includefilenames.isEmpty()){
                           sh "echo ${ZIP_NODE} && echo 'remove alraedy existing zip files' && rm -rf *.zip && zip -r ${includedfile} --exclude=*.git* --exclude=*/.* ${ZIP_NODE} . && chmod 777 ${ZIP_NODE}"
        
                       }else{
                            sh "echo ${ZIP_NODE} && echo 'remove alraedy existing zip files' && rm -rf *.zip && zip -r --exclude=*deployment-artifacts* --exclude=*postdeployment* ${ZIP_NODE} * && chmod 777 ${ZIP_NODE}" 
                       }
                       
             		} 
			}
		}
	}
}
