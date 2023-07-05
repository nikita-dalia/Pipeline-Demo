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
 
 def statusvalue
String includedfile="";

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
                    def includedFileObject=new File("${jsonIncludefilepath}")
                    def includedFileJSONObject=new JsonSlurperClassic().parse(includedFileObject)

                    def includedFilenamesString=includedFileJSONObject.filename

                  println("includedFilenamesString: " + includedFilenamesString)
                    String[] arrOfIncludedFilenames= includedFilenamesString.trim().split(",");
                    for(int i=0;i<arrOfIncludedFilenames.length;i++)
                    {
                        includedfile +=arrOfIncludedFilenames[i]+" "
                    }
                    println("Final included files:"+includedfile)
                }
            }
        }
    }
}
