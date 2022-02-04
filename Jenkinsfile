import groovy.json.JsonSlurperClassic

def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}
pipeline {
    agent any
    environment {
        NEXUS_USER         = credentials('NEXUS-USER')
        NEXUS_PASSWORD     = credentials('NEXUS-PASS')
    }
    parameters {
        choice(
            name:'compileTool',
            choices: ['Maven', 'Gradle'],
            description: 'Seleccione herramienta de compilacion'
        )
    }
    stages {
        stage('Pipeline') {
            steps {
                script {
                    switch (params.compileTool)
                    {
                        case 'Maven':
                            def ejecucion = load 'maven.groovy'
                            ejecucion.call()
                            break
                        case 'Gradle':
                            def ejecucion = load 'gradle.groovy'
                            ejecucion.call()
                            break
                    }
                }
            }
            post {
                always {
                    sh "echo 'fase always executed post'"
                    slackSend channel: 'sección1-grupo4', message: 'FINALIZA PROCESO'
                }
                success {
                    sh "echo 'fase success'"
                    slackSend color: 'good', channel: 'sección1-grupo4', message: "[Grupo2] [${JOB_NAME}] [${BUILD_TAG}] Ejecucion Exitosa", teamDomain: 'dipdevopsusac-tr94431'
                }
                failure {
                    sh "echo 'fase failure'"
                    slackSend color: 'danger', channel: 'sección1-grupo4', message: "[Grupo2] [${env.JOB_NAME}] [${BUILD_TAG}] Ejecucion fallida en stage [${env.TAREA}]", teamDomain: 'dipdevopsusac-tr94431'
                }
            }
        }
    }
}
