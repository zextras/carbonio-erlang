// SPDX-FileCopyrightText: 2022 Zextras <https://www.zextras.com>
//
// SPDX-License-Identifier: AGPL-3.0-only

library(
    identifier: 'jenkins-lib-common@v4.10.10',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        credentialsId: 'jenkins-integration-with-github-account',
        remote: 'git@github.com:zextras/jenkins-lib-common.git',
    ])
)

properties(defaultPipelineProperties())

pipeline {
    agent {
        node {
            label 'zextras-v1'
        }
    }

    environment {
        FAILURE_EMAIL_RECIPIENTS = 'smokybeans@zextras.com'
    }

    options {
        skipDefaultCheckout()
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timeout(time: 3, unit: 'HOURS')
    }

    stages {
        stage('Setup') {
            steps {
                checkout scm
                script {
                    gitMetadata()
                }
            }
        }

        stage('Build deb/rpm') {
            steps {
                echo 'Building deb/rpm packages'
                buildStage(
                    addCarbonioRepos: true,
                    parallelBuilds: true,
                    prepare: true,
                )
                buildStage(
                    addCarbonioRepos: true,
                    architecture: 'aarch64',
                    buildFlags: ' --only carbonio-erlang ',
                    distros: ['ubuntu-jammy'],
                    parallelBuilds: false,
                    prepare: true,
                )
            }
            post {
                failure {
                    script {
                        if ("main".equals(env.BRANCH_NAME)) {
                            sendFailureEmail(STAGE_NAME)
                        }
                    }
                }
            }
        }

        stage('Upload artifacts') {
            when {
                expression { return uploadStage.shouldUpload() }
            }
            tools {
                jfrog 'jfrog-cli'
            }
            steps {
                uploadStage()
                uploadStage(
                    architecture: 'aarch64',
                    distros: ['ubuntu-jammy'],
                )
            }
            post {
                failure {
                    script {
                        if ("main".equals(env.BRANCH_NAME)) {
                            sendFailureEmail(STAGE_NAME)
                        }
                    }
                }
            }
        }
    }
}

void sendFailureEmail(String step) {
    String commitInfo = sh(
        script: 'git log -1 --pretty=tformat:\'<ul><li>Revision: %H</li><li>Title: %s</li><li>Author: %ae</li></ul>\'',
        returnStdout: true
    )
    emailext body: """\
        <b>${step.capitalize()}</b> step has failed on trunk.<br /><br />
        Last commit info: <br />
        ${commitInfo}<br /><br />
        Check the failing build at the <a href=\"${BUILD_URL}\">following link</a><br />
    """,
    subject: "[ERLANG TRUNK FAILURE] Trunk ${step} step failure",
    to: FAILURE_EMAIL_RECIPIENTS
}
