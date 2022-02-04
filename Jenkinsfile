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
    stages {
        stage("Pipeline"){
            steps {
                script{
                    stage("Paso 1: Build && Test") {
                        sh "echo 'Build && Test!'"
                        sh "gradle clean build"
                    // code
                    }
                    stage("Paso 2: Sonar - Análisis Estático") {
                        sh "echo 'Análisis Estático!'"
                        withSonarQubeEnv('sonarqube') {
                            sh "echo 'Calling sonar by ID!'"
                            // Run Maven on a Unix agent to execute Sonar.
                            sh './gradlew sonarqube -Dsonar.projectKey=ejemplo-gradle -Dsonar.java.binaries=build'
                        }
                    }
                    stage('Paso 3: Curl Springboot Gradle sleep 20') {
                        sh "gradle bootRun&"
                        sh "sleep 20 && curl -X GET 'http://localhost:8082/rest/mscovid/test?msg=testing'"
                    }
                    stage("Paso 4: Subir Nexus"){
                        nexusPublisher nexusInstanceId: 'nexus',
                        nexusRepositoryId: 'devops-usach-nexus',
                        packages: [
                            [$class: 'MavenPackage',
                                mavenAssetList: [
                                    [classifier: '',
                                    extension: 'jar',
                                    filePath: 'build/libs/DevOpsUsach2020-0.0.1.jar'
                                ]
                            ],
                                mavenCoordinate: [
                                    artifactId: 'DevOpsUsach2020',
                                    groupId: 'com.devopsusach2020',
                                    packaging: 'jar',
                                    version: '0.0.1-GRADLE-MS-ICLAB'
                                ]
                            ]
                        ]
                    }
                    stage("Paso 5: Descargar Nexus") {
                        sh ' curl -X GET -u $NEXUS_USER:$NEXUS_PASSWORD "http://nexus:8081/repository/devops-usach-nexus/com/devopsusach2020/DevOpsUsach2020/0.0.1-GRADLE-MS-ICLAB/DevOpsUsach2020-0.0.1-GRADLE-MS-ICLAB.jar" -O'
                    }
                    stage("Paso 6: Levantar Artefacto Jar"){
                        sh 'nohup bash java -jar DevOpsUsach2020-0.0.1-GRADLE-MS-ICLAB.jar & >/dev/null'
                    }
                    stage("Paso 7: Testear Artefacto - Dormir(Esperar 20sg) "){
                       sh "sleep 20 && curl -X GET 'http://localhost:8082/rest/mscovid/test?msg=testing'"
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
                    slackSend color: 'good', channel: 'sección1-grupo4',message: "[Grupo2][${BRANCH_NAME}][${JOB_NAME}] [${BUILD_TAG}] Ejecucion Exitosa --- [${env.JOB_NAME}] [${BUILD_TAG}] Ejecucion fallida en stage [${env.TAREA}]", teamDomain: 'dipdevopsusac-tr94431'
                }
                failure {
                    sh "echo 'fase failure'"
                    slackSend color: 'danger', channel: 'sección1-grupo4',message: "[Grupo2] [${env.JOB_NAME}] [${BUILD_TAG}] Ejecucion fallida en stage [${env.TAREA}]", teamDomain: 'dipdevopsusac-tr94431'
                }
            }
        }
    }
}