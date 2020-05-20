def variants = ["debug", "release"]

pipeline {
    agent{
        label 'automation_node'
    }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-1.8-openjdk"
        REPO = "https://github.com/nogala/android.git"
        WORKANDROID = "android"
        PROJECTKEY = "androidTest"
        APPID= "FIREBASE_APP_ID"
    }

    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '5')
        preserveStashes buildCount: 5
    }

    parameters {

        booleanParam defaultValue: false,
                description: 'Clean workspace',
                name: 'CLEAN'

        booleanParam defaultValue: true,
                description: 'Show all the commands for debug pipeline',
                name: 'DEBUG'

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
                    say.simple("Run pipeline for debug: ${params.DEBUG}")
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
                    say("Stage ${STAGE_NAME}")
                    if (params.CLEAN) {
                        if (params.DEBUG) say.simple("Cleaning workspace")
                        cleanWs()
                    }
                    getRepo(WORKANDROID,REPO, params.BRANCH, params.CREDENTIAL_GITHUB)
                    say("Stage OK")
                }
            }
        }

        stage('Config') {
            steps {
                script {
                    say("Stage ${STAGE_NAME}")
                    catchError(buildResult: 'SUCCESS', message: "Error on build", stageResult: 'NOT_BUILT') {
                        configCode(WORKANDROID, params.BUILD_NUMBER)
                    }
                    say("Stage OK")
                }
            }
        }

        stage("Build and Test") {
            parallel(){

                stage('Build') {
                    steps {
                        catchError(buildResult: 'SUCCESS', message: "Error on build", stageResult: 'NOT_BUILT') {
                            script {
                                say("Stage ${STAGE_NAME}")
                                buildCode(WORKANDROID, variants)
                                say("Stage OK")
                            }
                        }
                    }
                    agent{
                        docker {
                            args '-u root -v build:/out/apks'
                            image 'nogala/androidtest:latest'
                            label 'androidDocker'
                            reuseNode true
                        }
                    }
                }

                stage('SonarQube Scanner') {
                    steps {
                        catchError(buildResult: 'SUCCESS', message: "Error on Test Sonar", stageResult: 'NOT_BUILT') {
                            script {
                                say("Stage ${STAGE_NAME}")
                                testSonar(WORKANDROID, PROJECTKEY)
                                say("Stage OK")
                            }
                        }
                    }
                    agent{
                        docker {
                            args '-u root -v build:/out/apks'
                            image 'nogala/androidtest:latest'
                            label 'androidDocker'
                            reuseNode true
                        }
                    }
                }

            }
        }

        stage('Distribute phase') {
            options {
                timeout(time: 1, unit: 'HOURS')
            }
            input {
                message "Distribuite to firebase?"
                parameters {
                    booleanParam(name: 'DEPLOY_FIREBASE',  defaultValue: true, description: "Check de box if want to deploy app to firebase ?")
                }
            }
            steps {
                script{
                    say("Stage ${STAGE_NAME}")
                    say.debug("The response is: ${DEPLOY_FIREBASE}")
                    if(DEPLOY_FIREBASE=='true') {
                        deployFirebase(WORKANDROID, variants, APPID)
                    }
                    else say.simple("Skip deploy phase")
                    say("Stage OK")
                }
            }
        }

        stage('Clean workspace'){
            steps{
                script {
                    say("Stage ${STAGE_NAME}")
                    if(params.CLEAN) {
                        cleanWs()
                    }
                    say("Stage OK")
                }
            }
        }

    }

}

