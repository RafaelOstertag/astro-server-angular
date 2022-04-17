pipeline {
    agent {
        label 'java&&nodejs'
    }

    triggers {
        pollSCM '@hourly'
    }

    tools {
        maven 'Latest Maven'
    }

    parameters {
        string defaultValue: 'none', description: 'Version to deploy', name: 'VERSION', trim: true
    }

    options {
        ansiColor('xterm')
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '15')
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Build Package and deploy') {
            when {
                allOf {
                    expression { return params.VERSION != 'none' }
                    expression { return params.VERSION != '' }
                }
            }

            environment {
                NPM_VERSION = sh returnStdout: true, script: 'echo $VERSION | sed -e "s/-SNAPSHOT\\$/-dev.$BUILD_NUMBER/"'
            }

            steps {
                configFileProvider([configFile(fileId: '4f3d0128-0fdd-4de7-8536-5cbdd54a8baf', variable: 'MAVEN_SETTINGS_XML')]) {
                    sh 'mvn -B -s "$MAVEN_SETTINGS_XML" -Dastro-server-api.version=${VERSION} -Dpackage.version=${NPM_VERSION} clean package'
                }
                configFileProvider([configFile(fileId: 'e5293454-b063-4d53-be2d-ecb485e4f660', targetLocation: 'target/npm-astro-server/.npmrc')]) {
                    sh 'mvn -B exec:exec@publish-astro-server-angular-package'
                }
                configFileProvider([configFile(fileId: 'e5293454-b063-4d53-be2d-ecb485e4f660', targetLocation: 'target/npm-catalog-fetcher/.npmrc')]) {
                    sh 'mvn -B exec:exec@publish-astro-server-catalog-fetcher-angular-package'
                }

            }
        }
    }

    post {
        unsuccessful {
            mail to: "rafi@guengel.ch",
                    subject: "${JOB_NAME} (${BRANCH_NAME};${env.BUILD_DISPLAY_NAME}) -- ${currentBuild.currentResult}",
                    body: "Refer to ${currentBuild.absoluteUrl}"
        }
    }
}
