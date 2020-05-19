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
        REPO= "https://github.com/nogala/android.git"
        WORKANDROID= "android"
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

        choice choices: ['TEST', 'STAGING', 'DEPLOY', 'RELEASE'],
                description: 'Type of build: Test only, Test and Staging on nexus or Deploy to Firebase, Release to store',
                name: 'BUILD_TYPE'

        string defaultValue: 'master',
                description: 'Branch of code to build',
                name: 'BRANCH'

        string defaultValue: '1',
                description: 'Build number',
                name: 'BUILD_NUMBER'

        credentials credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl',
                defaultValue: 'TodoPagoGit',
                description: 'User of gitHub',
                name: 'CREDENTIAL_GITHUB',
                required: true
    }

    stages {

        stage('Read Parameters phase') {
            steps {
                script{
                    say ("Start ${STAGE_NAME}")
                    say.simple("Clean workspace: ${params.CLEAN}")
                    say.simple("Type of build: ${params.BUILD_TYPE}")
                    say.simple("Repository to build: ${REPO}")
                    say.simple("Branch: ${params.BRANCH}")
                    say.simple("Build Number: ${params.BUILD_NUMBER}")
                    say("Stage OK")
                }
            }
        }

        stage('Checkout') {
            steps {
                script{
                    if (params.CLEAN){
                        cleanWs()
                    }
                    say("Stage ${STAGE_NAME}")
                    getRepo(WORKANDROID,REPO, params.BRANCH, params.CREDENTIAL_GITHUB)
                    say("Stage OK")
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

