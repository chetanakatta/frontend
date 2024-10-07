pipeline {

    agent {
        label 'AGENT-1' // our agent name
    }

    options {

        //after particular time job will be failed (timeout counter starts before agent is allocated)
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds() //disables multiple executions
        ansiColor('xterm')

    }

    environment {

        def appVersion = ''  //variable declaration
        nexusUrl = 'nexus.expense.fun:8081'

    }

    stages {

        stage ('read the version') { 
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version: $appVersion"
                }
            }
        }

        stage ('Build') {
            steps {
                sh """
                zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                ls -ltr
                """
            }
        }

        stage ('Nexus Artifact Upload') {
            steps {
                script {
                    nexusArtifactUploader(
                       nexusVersion: 'nexus3',
                       protocol: 'http',
                       nexusUrl: "${nexusUrl}",
                       groupId: 'com.expense',
                       version: "${appVersion}",
                       repository: "frontend",
                       credentialsId: 'nexus-auth',
                       artifacts: [
                        [artifactId: "frontend",
                        classifier: '',
                        file: "frontend-" + "${appVersion}" + '.zip',
                        type: 'zip']
                       ]

                    )

                }
            }
        }

        stage ('Deploy') {
            steps {
                script {
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'frontend-deploy', parameters: params, wait: false
                }

            }
        }

    }    
    post { //useful as alert for success or failure

        always {
            echo 'I will always say hello'
            deleteDir()    //to delete workspace after build
        }

        success {
            echo 'I will run when pipeline is success'
        }

        failure {
            echo 'I will run when pipeline is failure'
            // we configure with slack, when failed we get messege
        }
    }
}