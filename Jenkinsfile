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
 String includedfile
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
					
					println(jsonIncludefilepath)
					println(ZIP_NODE)
				}

			}
		}
		stage("Prepare Package"){
			steps{
				echo "Preparing Package Stage..."
				script {
                     
                     try{
                         println(1)
                         def inputeIncludeFile = new File("${jsonIncludefilepath}")
						 println(2)
                         def InputIncludeJSON = new JsonSlurper().parse(inputeIncludeFile)
			     			println(3)
                         includefilenames = InputIncludeJSON.filename
			     		 println(4)
                         String[] arrOfIncludedfiles = includefilenames.split(","); 
			     		 println(5)
                         for(int i=0; i< arrOfIncludedfiles.length; i++)
                           {
                                def includedfileoutput  = "--include"+"=*"+arrOfIncludedfiles[i]-'['-']'-' '+"*";
                    
                                   includedfile = includedfile+includedfileoutput+" "
                                 //  println("data["+i+"] : "+arrOfIncludedfiles[i])
                          }
			     		 println(6)
                          
                        }catch(Exception e) {
					 println(7)
                       }
                	 println(8)
	                      //  println(includefilenames)
                      if(includefilenames != null && !includefilenames.isEmpty()){
						 println(9)
						 println(ZIP_NODE)
						 echo "gello"
						 echo "${ZIP_NODE}"
						 echo 'remove alraedy existing zip files'
						 bat "del /S /Q *.zip"
						 println(14)
						 println(includedfile)
						 bat "powershell.exe -Command Compress-Archive -Path '.\\rock-business-actions_thing_admin_pmi.json' -DestinationPath main.zip -CompressionLevel Optimal; icacls 'main.zip' /grant Everyone:F"
			      		 //sh "zip -r ${includedfile} --exclude=*.git* --exclude=*/.* ${ZIP_NODE} . && chmod 777 ${ZIP_NODE}"
        				 println(10)
                       }else{
			     		 println(11) 
						 //sh "echo ${ZIP_NODE} && echo 'remove alraedy existing zip files' && rm -rf *.zip && zip -r --exclude=*deployment-artifacts* --exclude=*postdeployment* ${ZIP_NODE} * && chmod 777 ${ZIP_NODE}" 
						 println(12)
		      			}
						 println(13)
				} 
			}
		}
		stage("Creating folder"){
			steps{
				script{
					def jsonobject = "{\"binaryStreamObject\":{\"id\":\"guid\",\"type\":\"seedDataStream\",\"properties\":{\"objectKey\":\"main.zip\",\"originalFileName\":\"main.zip\"}}}"
                    def post = new URL("https://etronds.riversand.com/api/binarystreamobjectservice/prepareUpload").openConnection();
                    def message = '{"message":"this is a message"}'
                    post.setRequestMethod("POST")
                    post.setDoOutput(true)
                    post.setRequestProperty("Content-Type","application/json")
                    post.setRequestProperty("x-rdp-version","8.1")
                    post.setRequestProperty("x-rdp-tenantId","etronds") 
                    post.setRequestProperty("x-rdp-clientId","rdpclient")
                    
                    post.setRequestProperty("x-rdp-userId","etronds.systemadmin@riversand.com")
                    
                    post.setRequestProperty("x-rdp-userRoles","systemadmin")
                    
                    post.setRequestProperty("auth-client-id","j29DTHa7m7VHucWbHg7VvYA75pUjBopS")
                    
                    post.setRequestProperty("auth-client-secret","J7UaRWQgxorI8mdfuu8y0mOLqzlIJo2hM3O4VfhX1PIeoa7CYVX_l0-BnHRtuSWB")
                    post.connect();

					OutputStreamWriter out = new OutputStreamWriter(post.getOutputStream());
                    out.write(jsonobject);
                    out.close();
                    def statuscode1 = post.getResponseCode();
                    String outputObj = post.getInputStream().getText();
                    println("StatusCode="+statuscode1)
                    println("outputObj="+outputObj)
                    Map jsonContent = (Map) new JsonSlurper().parseText(outputObj)
                    String data1 = jsonContent.response.binaryStreamObjects.data
                    String[] arrOfStr = data1.split(","); 
                    // println("Number of substrings: "+arrOfStr.length);
                         for(int i=0; i< arrOfStr.length; i++)
                           {
                              String[] arrOfurl = arrOfStr[i].split("uploadURL=")
                              for(int p=1; p< arrOfurl.length; p++){
                              fileuploadUrl = arrOfurl[p]-"}}]"	
                               println("data["+p+"] : "+arrOfurl[p])
                          }
                         }
                       println(fileuploadUrl)
				}
			}
		}
		stage('Deploying folder'){
            steps{
                script{
                     echo "====Deploying folder====="
                    /*sh """ curl -v -X PUT '${fileuploadUrl}' \
                     --header 'x-ms-meta-x_rdp_userroles: ${env.xrdpuserRoles}' \
                     --header 'x-ms-meta-x_rdp_tenantid: ${params.TENANT_ID}' \
                     --header 'x-ms-meta-originalfilename: ${ZIP_NODE}' \
                     --header 'x-ms-blob-content-disposition: attachment; filename=${ZIP_NODE}' \
                     --header 'x-ms-meta-type: ${env.xmsmetatype}' \
                     --header 'x-ms-meta-x_rdp_clientid: ${params.XRDP_CLIENT_ID}' \
                     --header 'x-ms-meta-x_rdp_userid: ${params.USER_ID}' \
                     --header 'x-ms-meta-binarystreamobjectid: ${env.xmsmetabinarystreamobjectid}' \
                     --header 'x-ms-blob-type: ${env.xmsblobtype}' \
                     --header 'Content-Type: application/zip' \
                     --data-binary '@${zipfilepath}'"""*/
					 /*bat '''
					 	curl -v -X PUT %fileuploadUrl% ^
						--header "x-ms-meta-x_rdp_userroles: systemadmin" ^
						--header "x-ms-meta-x_rdp_tenantid: etronds" ^
						--header "x-ms-meta-originalfilename: main.zip" ^
						--header "x-ms-blob-content-disposition: attachment; filename=main.zip" ^
						--header "x-ms-meta-type: seedDataStream" ^
						--header "x-ms-meta-x_rdp_clientid: rdpclient" ^
						--header "x-ms-meta-x_rdp_userid: etronds.systemadmin@riversand.com" ^
						--header "x-ms-meta-binarystreamobjectid: guid" ^
						--header "x-ms-blob-type: guid" ^
						--header "Content-Type: application/zip" ^
						--data-binary @%zipfilepath%
					 '''*/
					 println(fileuploadUrl)
					 println(zipfilepath)
					 def encodedFileUploadUrl = URLEncoder.encode(fileuploadUrl, "UTF-8")
					 bat """
							curl -v -X PUT ^ --url "${fileuploadUrl}" ^
								--header 'x-ms-meta-x_rdp_userroles: systemadmin ^
								--header 'x-ms-meta-x_rdp_tenantid: etronds' ^
								--header 'x-ms-meta-originalfilename: main.zip' ^
								--header 'x-ms-blob-content-disposition: attachment; filename=main.zip' ^
								--header 'x-ms-meta-type: seedDataStream' ^
								--header 'x-ms-meta-x_rdp_clientid: rdpclient' ^
								--header 'x-ms-meta-x_rdp_userid: etronds.systemadmin@riversand.com' ^
								--header 'x-ms-meta-binarystreamobjectid: guid' ^
								--header 'x-ms-blob-type: BlockBlob' ^
								--header 'Content-Type: application/zip' ^
								--data-binary '@${zipfilepath}'
							"""
                  }    
            }
        }

	}
}
