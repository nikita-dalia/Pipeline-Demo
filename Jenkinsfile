import groovy.json.JsonSlurper
import groovy.json.*
import java.io.OutputStreamWriter
import java.lang.String
import groovy.transform.Field
import jenkins.model.*
import java.io.FileReader;
import java.util.Iterator;
import java.util.Map
import groovy.io.FileType
import groovy.json.JsonSlurperClassic
import hudson.model.*

def NAME_ZIPFILE
def ZIP_WORKFLOW
def jsonResponse
String fileuploadUrl
String postuploadfileuploadUrl
def list = []
String taskID
String path_zipfile
String zip_workflowfilepath
String includefilenames
String filedata
def path_filestobedeployed 
def postdeploymentfilepath
def statuscode
def objectstatus
def objectsecondstatus
String statusDetail1msg = ""
def status_final
def statusvalue
String includedfile = ""
def tenant="DS"
def tenantstobeexcluded

@NonCPS
def makeApiCallAndGetResponse(String taskID) {
    def post = new URL("https://etronds.riversand.com/api/requesttrackingservice/get").openConnection() as HttpURLConnection
    def requestData = '{"params":{"query":{"id":"' + taskID + '","filters":{"typesCriterion":["tasksummaryobject"]}},"fields":{"attributes":["_ALL"],"relationships":["_ALL"]},"options":{"maxRecords":1000}}}'
    def message = '{"message":"this is a message"}'

    post.setRequestMethod("POST")
    post.setDoOutput(true)
    post.setRequestProperty("Content-Type", "application/zip")
    post.setRequestProperty("x-rdp-version", "8.1")
    post.setRequestProperty("x-rdp-tenantId", "etronds")
    post.setRequestProperty("x-rdp-clientId", "rdpclient")
    post.setRequestProperty("x-rdp-userId", "etronds.systemadmin@riversand.com")
    post.setRequestProperty("x-rdp-userRoles", "systemadmin")
    post.setRequestProperty("auth-client-id", "j29DTHa7m7VHucWbHg7VvYA75pUjBopS")
    post.setRequestProperty("auth-client-secret", "J7UaRWQgxorI8mdfuu8y0mOLqzlIJo2hM3O4VfhX1PIeoa7CYVX_l0-BnHRtuSWB")

    post.connect()

    // Write the request data to the output stream
    def outputStream = post.getOutputStream() as OutputStream
    outputStream.write(requestData.getBytes("UTF-8"))
    outputStream.flush()
    outputStream.close()

    // Get the response from the input stream
    def responseCode = post.getResponseCode()
    def inputStream = (responseCode == HttpURLConnection.HTTP_OK) ? post.getInputStream() : post.getErrorStream()
    def reader = new BufferedReader(new InputStreamReader(inputStream))
    def responseBuilder = new StringBuilder()
    String line
    while ((line = reader.readLine()) != null) {
        responseBuilder.append(line)
    }
    def responsess = responseBuilder.toString()

    reader.close()
    inputStream.close()
    post.disconnect()

    return responsess
}

pipeline {
    agent any

    stages {
        stage('Path Variables Initialization') {
            steps {
                script {
                    NAME_ZIPFILE = "${env.BRANCH_NAME}.zip"
                    ZIP_WORKFLOW = "postdeployment.zip"
                    path_filestobedeployed = "${env.WORKSPACE}/deployment-artifacts/filestobedeployed.json"
                    path_zipfile = "${env.WORKSPACE}/${NAME_ZIPFILE}"
                    zip_workflowfilepath = "${env.WORKSPACE}/${ZIP_WORKFLOW}"

                    println("NAME_ZIPFILE: " + NAME_ZIPFILE)
                    println("ZIP_WORKFLOW: " + ZIP_WORKFLOW)
                    println("path_filestobedeployed: " + path_filestobedeployed)
                    println("path_zipfile: " + path_zipfile)
                    println("zip_workflowfilepath: " + zip_workflowfilepath)
                }
            }
        }

        stage('Zipping Files') {
            steps {
                script {
                    def includedFileObject = new File("${path_filestobedeployed}")
                    def includedFileJSONObject = new JsonSlurperClassic().parse(includedFileObject)

                    def includedFilenamesString = includedFileJSONObject.filename

                    println("includedFilenamesString: " + includedFilenamesString)

                    if (includedFilenamesString.size() > 1) {
                        // The string contains a comma
                        for (int i = 0; i < includedFilenamesString.size(); i++) {
                            if (i >= 1) {
                                includedfile += ","
                            }
                            includedFilenamesString[i].trim().replaceAll("\\[|\\]", "")
                            includedfile += "'${env.WORKSPACE}" + "\\" + includedFilenamesString[i] + "'"
                        }
                        println("Final included files: " + includedfile)
                    } else {
                        // The string does not contain a comma
                        // Assuming includedFilenamesString is an ArrayList, extract a string element
                        String filenamesString = includedFilenamesString.get(0)

                        // Trim leading and trailing whitespace, and remove square brackets
                        filenamesString = filenamesString.trim().replaceAll("\\[|\\]", "")
                        includedfile = "'${env.WORKSPACE}" + "\\" + filenamesString + "'"
                        println("No comma present" + includedfile)
                    }

                    println("Final included files: " + includedfile)
                    println("Filenames processed successfully... Moving to zipping..")

                    if(tenant=="DS"){
                        tenantstobeexcluded=['FS','PROD']
                    }
                    else if(tenant=="FS"){
                        tenantstobeexcluded=['DS','PROD']
                    }
                    else{
                        tenantstobeexcluded=['FS','DS']
                    }


                    /*bat """
                    powershell.exe -Command "if (Test-Path '${path_zipfile}') { Remove-Item '${path_zipfile}' }"
                    powershell.exe -Command "Compress-Archive -Path @(${includedfile}) -DestinationPath '${path_zipfile}'"
                    """*/
                    // Convert the array to a comma-separated string to pass as an environment variable
                    def excludedFoldersArgs = tenantstobeexcluded.collect { "'${it}'" }.join(',')

                    bat """
                        powershell.exe -Command "if (Test-Path '${path_zipfile}') { Remove-Item '${path_zipfile}' }"
                        powershell.exe -Command "Compress-Archive -Path @(Get-ChildItem '${includedfile}' | Where-Object { $_.Name -NotIn @(${excludedFoldersArgs}) }) -DestinationPath '${path_zipfile}'"
                    """
                    if (!fileExists(path_zipfile)) {
                        error("Failed to create the zip file.")
                    } else {
                        println("Files zipped successfully...")
                    }
                }
            }
        }
        /*
        stage('Prepare Upload') {
            steps {
                script {
                    def jsonobject = "{\"binaryStreamObject\":{\"id\":\"guid\",\"type\":\"seedDataStream\",\"properties\":{\"objectKey\":\"${NAME_ZIPFILE}\",\"originalFileName\":\"${NAME_ZIPFILE}\"}}}"

                    def post = new URL("https://etronds.riversand.com/api/binarystreamobjectservice/prepareUpload").openConnection();
                    def message = '{"message":"this is a message"}'
                    post.setRequestMethod("POST")
                    post.setDoOutput(true)
                    post.setRequestProperty("Content-Type", "application/json")
                    post.setRequestProperty("x-rdp-version", "8.1")
                    post.setRequestProperty("x-rdp-tenantId", "etronds")
                    post.setRequestProperty("x-rdp-clientId", "rdpclient")
                    post.setRequestProperty("x-rdp-userId", "etronds.systemadmin@riversand.com")
                    post.setRequestProperty("x-rdp-userRoles", "systemadmin")
                    post.setRequestProperty("auth-client-id", "j29DTHa7m7VHucWbHg7VvYA75pUjBopS")
                    post.setRequestProperty("auth-client-secret", "J7UaRWQgxorI8mdfuu8y0mOLqzlIJo2hM3O4VfhX1PIeoa7CYVX_l0-BnHRtuSWB")
                    post.connect()

                    OutputStreamWriter out = new OutputStreamWriter(post.getOutputStream())
                    out.write(jsonobject)
                    out.close()
                    def statuscode1 = post.getResponseCode()
                    String outputObj = post.getInputStream().getText()
                    println("StatusCode=" + statuscode1)
                    println("outputObj=" + outputObj)
                    Map jsonContent = (Map) new JsonSlurper().parseText(outputObj)
                    String data1 = jsonContent.response.binaryStreamObjects.data
                    String[] arrOfStr = data1.split(",")
                    
                    for (int i = 0; i < arrOfStr.length; i++) {
                        String[] arrOfurl = arrOfStr[i].split("uploadURL=")
                        
                        for (int p = 1; p < arrOfurl.length; p++) {
                            fileuploadUrl = arrOfurl[p] - "}}]"
                            println("data[" + p + "]: " + arrOfurl[p])
                        }
                    }
                    
                    println(fileuploadUrl)
                }
            }
        }

        stage('Deploy Files') {
            steps {
                script {
                    echo "==== Deploying folder ===="

                    def encodedFileuploadUrl = fileuploadUrl.replaceAll('%', '%%')

                    bat """
                        curl -v -X PUT "${encodedFileuploadUrl}" ^
                        --header "x-ms-meta-x_rdp_userroles: systemadmin" ^
                        --header "x-ms-meta-x_rdp_tenantid: etronds" ^
                        --header "x-ms-meta-originalfilename: ${NAME_ZIPFILE}" ^
                        --header "x-ms-blob-content-disposition: attachment; filename=${NAME_ZIPFILE}" ^
                        --header "x-ms-meta-type: disposition" ^
                        --header "x-ms-meta-x_rdp_clientid: rdpclient" ^
                        --header "x-ms-meta-x_rdp_userid: etronds.systemadmin@riversand.com" ^
                        --header "x-ms-meta-binarystreamobjectid: guid" ^
                        --header "x-ms-blob-type: BlockBlob" ^
                        --header "Content-Type: application/zip" ^
                        --data-binary "@${path_zipfile}"
                    """
                }
            }
        }

        stage('Upload to Tenant'){
                steps{
                    script{
                        echo "====Deployment====="
                        def jsonobject = "{\"adminObject\":{\"id\":\"someguid\",\"type\":\"adminObject\",\"properties\":{\"flushConfig\":false,\"storageType\":\"stream\",\"objectKey\":\"${NAME_ZIPFILE}\",\"tenantId\":\"etronds\",\"retryCount\":1,\"sleepTime\":1000}}}"
                        def post = new URL("https://etronds.riversand.com/api/adminservice/deploytenantseed").openConnection();
                        def message = '{"message":"this is a message"}'
                        post.setRequestMethod("POST")
                        post.setDoOutput(true)
                        post.setRequestProperty("Content-Type","application/zip")
                        post.setRequestProperty("x-rdp-version","8.1")
                        
                        post.setRequestProperty("x-rdp-clientId","rdpclient")
                        
                        post.setRequestProperty("x-rdp-userId","etronds.systemadmin@riversand.com")
                        
                        post.setRequestProperty("x-rdp-userRoles","systemadmin")
                        
                        post.setRequestProperty("auth-client-id","j29DTHa7m7VHucWbHg7VvYA75pUjBopS")
                        
                        post.setRequestProperty("auth-client-secret","J7UaRWQgxorI8mdfuu8y0mOLqzlIJo2hM3O4VfhX1PIeoa7CYVX_l0-BnHRtuSWB")
                        OutputStreamWriter out = new OutputStreamWriter(post.getOutputStream());
                        out.write(jsonobject);
                        out.close();
                        post.getOutputStream().write(message.getBytes("UTF-8"));
                        def statuscode2 = post.getResponseCode();
                        def outputObj = post.getInputStream().getText();
                        println("statusCode=" +statuscode2)
                        Map jsonContent = (Map) new JsonSlurper().parseText(outputObj)
                        println(jsonContent)
                        def status = jsonContent.response.status
                        def totalRecords= jsonContent.response.totalRecords
                        taskID = jsonContent.response.statusDetail.taskId
                        println("Status="+status);
                        println("Taskid="+taskID)
                        println("TotalRecod="+totalRecords);

                    }    
            }
        }
        
        stage('Verify Upload') {
            steps {
                script {
                    def taskstatus = false
                    def responsess

                    while (!taskstatus) {
                        responsess = makeApiCallAndGetResponse(taskID)

                        // Process the response
                        println("task_mssage response: " + responsess)

                        node {
                            // Run non-serializable operations on the agent
                            def jsonSlurper = new groovy.json.JsonSlurper()
                            def jsonContent = jsonSlurper.parseText(responsess.trim())
                            def totalRecord = jsonContent.response.totalRecords

                            if (totalRecord == 1) {
                                objectstatus = jsonContent.response.requestObjects[0].data.attributes.status.values[0].value
                                println("=========== objecttttt=found====" + objectstatus)
                                if (objectstatus == "Completed" || objectstatus == "Completed with errors" || objectstatus == "Errored") {
                                    taskstatus = true
                                }
                            } else {
                                statusDetail1msg = jsonContent.response.statusDetail.messages[0].message
                                println("===========no objecttttt=====" + statusDetail1msg)
                            }
                        }

                        if (!taskstatus) {
                            sleep(30)
                        }
                    }
                }
            }
        }
         */

    }
}
