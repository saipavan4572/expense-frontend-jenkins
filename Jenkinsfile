pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        //timeout(time: 5, unit: 'SECONDS')
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()  // it will not allow concurrent builds
        ansiColor('xterm')
    }
    environment{
        def appVersion = '' //variable declaration
        nexusUrl = 'nexus.pspkdevops.online:8081'
        // region = "us-east-1"
        // account_id = "315069654700"
    }
    stages {

        // stage('test'){
        //     steps{
        //         sh """
        //         echo 'this is sample testing stage'
        //         ls -ltr

        //         """
        //     }
        // }


        stage('read the version'){
            steps{
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"
                }
            }
        }

        stage('Build'){
            steps{
                sh """
                zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                ls -ltr
                """
            }
        }
        ////ex: zip -q -r frontend-1.0.0.zip * -x Jenkinsfile -x frontend-1.0.0.zip
        // -q (quiet)is used to not to show the unnecessary console logs while performing zip action/task
        // -x is used to exclude that specific file from zip in that directory.

        stage('Nexus Artifact Upload'){
            steps{
                echo 'Hi, this is Nexus Artifact Uploading stage'
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "frontend",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: "frontend" ,
                            classifier: '',
                            file: "frontend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
                }
            }
        }

        stage('Deploy'){
         //   /* when{
         //       expression{
         //           params.deploy
         //       }
         //   } */
            steps{
                echo 'Hi, this is triggering CD job stage'
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'frontend-deploy', parameters: params, wait: false
                    // since the build job is in the same folder so we mentioned module name directly, if it is in different folder then we need to mention the folder path like: 'com/xyz/featurename'
                    // name given in jenkins server: "frontend-deploy"
                }
            }
        }

    }

    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()     // workspace has to be deleted after every build to avoid issues for next builds
        }
        success { 
            echo 'I will say Hello only when it is success!'
        }
        failure { 
            echo 'I will say Bye when it is failed!'
        }
    }

}