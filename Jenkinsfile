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
                    configCode(WORKANDROID, params.BUILD_NUMBER)
                    say("Stage OK")
                }
            }
        }

        stage("Build and Test") {
            parallel()
                stages {
                    stage('Build') {
                        steps {
                            catchError(buildResult: 'SUCCESS', message: "Error on build", stageResult: 'NOT_BUILT') {
                                script {
                                    say("Stage ${STAGE_NAME}")
                                    def variants = ["debug", "release"]
                                    buildCode(WORKANDROID, variants)
                                    say("Stage OK")
                                }
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
                    }
                }
            }

        stage('Decide Upload to Firebase') {
            options {
                timeout(time: 1, unit: 'HOURS')
            }
            input id: 'QUESTION',
                    message: 'Distribuite',
                    ok: 'true',
                    parameters: [
                            booleanParam(defaultValue: false,
                                    description: 'Deploy app to firebase ?',
                                    name: 'DEPLOY_FIREBASE')],
                    submitterParameter: 'DEPLOY'
            steps {
                script {
                    say("Stage ${STAGE_NAME}")
                    say("The response is: ${QUESION.DEPLOY}")
                    say("Stage OK")
                }
            }
        }

        stage('Distribute phase') {
            steps {
                catchError(buildResult: 'SUCCESS', message: "Error on deploy Variant ${it}", stageResult: 'NOT_BUILT') {
                    script {
                        say("Stage ${STAGE_NAME}")
                        if(params.BUILD_TYPE == "DEPLOY" || params.BUILD_TYPE == "RELEASE") {
                            def variants = ["debug", "release"]
                            deployFirebase(WORKANDROID, variants, APPID)
                        }
                        else say.simple("Skip deploy phase")
                        say("Stage OK")
                    }
                    }
                }
            }

        stage('Clean workspace'){
            steps{
                script {
                    say("Stage ${STAGE_NAME}")
                    if(params.CLEAN) cleanWs()
                    say("Stage OK")
                }
            }
        }

    }

}

