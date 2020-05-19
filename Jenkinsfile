libraries {
    /*
        TODO configurate shared libraries first.
     */
    lib('androidLibs@master')
}

pipeline {
    agent{
        label 'automation_node'
    }
    environment {
        /* TODO Save here all the environment values
            example:
                JAVA_HOME = "/usr/lib/jvm/java-1.8-openjdk"
        * */
        JAVA_HOME = "/usr/lib/jvm/java-1.8-openjdk"
    }
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '5')
        preserveStashes buildCount: 5
    }

    parameters {
        /*
            TODO What are the parameters to use
                example:
                booleanParam defaultValue: true,
                description: 'Clean workspace',
                name: 'CLEAN'
         */
        booleanParam defaultValue: true,
                description: 'Clean workspace',
                name: 'CLEAN'
    }

    stages {

        stage('Read Parameters phase') {
            steps {
                script{
                    say ("Start ${STAGE_NAME}")
                    say.simple("Clean workspace: ${params.CLEAN}")
                    /*
                        TODO Read all the parameters provided:
                            example:
                                The current parameters are:
                                Clean workspace: ${params.CLEAN}
                     */
                    say("Stage OK")
                }
            }
        }

        stage('Checkout') {
            steps {
                script{
                    echo "*********************************** Stage ${STAGE_NAME} ***********************************"
                    cleanWs()
                    /*
                        TODO add shared function here
                            example:
                            getRepo(workspace, repo, branch, credentialID)
                     */
                    echo "*********************************** Stage OK ***********************************"
                }
            }
        }

        stage('Config') {
            steps {
                script {
                    echo "*********************************** Stage ${STAGE_NAME} ***********************************"
                    /*
                        TODO add configuration functions
                            example:
                                replaceString(sourceFile, toSearch, replacement)
                            on all files needed ...
                     */
                    echo "*********************************** Stage OK ***********************************"
                }
            }
        }

        stage("Build and Test") {
            parallel() {
                stage('Build') {
                    steps {
                        catchError(buildResult: 'SUCCESS', message: "Error on build", stageResult: 'NOT_BUILT') {
                            script {
                                echo "*********************************** Stage ${STAGE_NAME} ***********************************"
                                /*
                                    TODO add configuration functions
                                        example:
                                            buildCode(workspace, command, variant, ....)
                                        on all files needed ...
                                 */
                                echo "*********************************** Stage OK ***********************************"

                            }
                        }
                    }
                }

                stage('SonarQube Scanner') {
                    steps {
                        catchError(buildResult: 'SUCCESS', message: "Error on Test Sonar", stageResult: 'NOT_BUILT') {
                            script {
                                echo "*********************************** Stage ${STAGE_NAME} ***********************************"
                                /*
                                    TODO withSonarQubeEnv(SonarServer)
                                        Note: configurate Sonarserver first
                                 */
                                echo "*********************************** Stage OK ***********************************"
                            }
                        }
                    }
                }
            }
        }

        stage('Decide Upload to Firebase') {
            steps {
                echo "*********************************** Stage ${STAGE_NAME} ***********************************"
            }
            options {
                timeout(time: 1, unit: 'HOURS')
            }
            input {
                message 'Approve Distribute?'
                parameters {
                    booleanParam defaultValue: true, description: 'Distribuite to Firebase ?', name: 'DEPLOY_FIREBASE'
                }
            }
        }

        stage('Distribute phase') {
            steps {
                catchError(buildResult: 'SUCCESS', message: "Error on deploy Variant ${it}", stageResult: 'NOT_BUILT') {
                    script {
                        echo "*********************************** Stage ${STAGE_NAME} ***********************************"
                        /*
                            TODO add deploy functions
                         */
                        echo "*********************************** Stage OK ***********************************"
                    }
                    }
                }
            }

        stage('Clean workspace'){
            steps{
                script {
                    if(params.CLEAN){
                        cleanWs()
                    }
                }
            }
        }
    }
}

