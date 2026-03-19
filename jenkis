def send_email(name, id, status, output1){
    echo """ Job: ${name} id: ${id} has status ${status} """
    echo """ ${output1}"""
}
def output=""
pipeline {
    agent any
    parameters {
    string (
        name: 'ARTIFACT_NAME',
        defaultValue: 'pipeline_default.zip', 
        description: 'neki opis'
    )
    booleanParam (
        defaultValue: true, 
        description: 'run test if true', 
        name: 'RUN_TEST'
        )
    booleanParam (
         name: 'FAIL_PIPELINE',
        description: "pada ako je true"
        )
 
  booleanParam (
      defaultValue: true, 
    description: 'if true send eamail', 
    name: 'SEND_EMAIL'
    )  
    }
    stages {
        stage("Download"){
            steps{ 
                cleanWs()
                dir("pipeline"){
                    git(
                        branch: "pipeline",
                        url: "https://github.com/KLevon/jenkins-course"
                        )
                }
                rtDownload (
                    serverId:'jfrog1',
                    spec: '''{
                        "files" : [
                            {
                            "pattern" : "generic-local/libraries/printer.zip",
                            "target" : "./",
                            "flat": "true"
                            }
                        ]
                        }'''
                )
                unzip(
                    zipFile:"printer.zip",
                    dir:"pipeline"
                    )
            }
        }
        stage("Build"){
            steps{
                echo(message:"nesto build")
                withCredentials(
                    [usernamePassword(credentialsId: "ana", passwordVariable: "psw", usernameVariable: "usr" )]){
                        echo(message:"${usr}") 
                    }
                bat(
                    script: '''
                    cd pipeline
                    Makefile.bat ''' )
            }
        }
        stage("Test"){
            when {
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps{
                script{
                    def array=["printer", "scanner", "main"]
                    
                    for (element in array) {
                        output += 
                        bat ( script: """
                            cd pipeline 
                            Tests.bat ${element}
                        """ , returnStdout: true).trim() 
                    }
                }
            }
        }
        stage("Publish"){
            steps{
                script{
                    zip(
                        zipFile: "${params.ARTIFACT_NAME}",
                        archive: true,
                        dir: "pipeline",
                        glob: ""
                        )
                }
                 rtUpload (
                    serverId:'jfrog1',
                    spec: """{
                        "files" : [
                            {
                            "pattern" : "${params.ARTIFACT_NAME}",
                            "target" : "generic-local/release/andjelac/${env.BUILD_ID}/"
                            }
                        ]
                        }"""
                )
                echo(message:"nesto")
                script{
                    if(params.FAIL_PIPELINE==true){
                    error("greska") 
                    }
                }
                
                
            }
        }
}
    post {
        failure{
            script{
                if(SEND_EMAIL){
                    send_email(env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult, output)
                }
            }
        }
    }
}
