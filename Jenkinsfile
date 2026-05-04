// SPDX-FileCopyrightText: 2022 Zextras <https://www.zextras.com>
//
// SPDX-License-Identifier: AGPL-3.0-only

library(
    identifier: 'jenkins-lib-common@1.7.1',
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
                withCredentials([usernamePassword(credentialsId: 'artifactory-jenkins-gradle-properties-splitted',
                        passwordVariable: 'SECRET',
                        usernameVariable: 'USERNAME')]) {
                    buildStage([
                        prepare: true,
                        prepareFlags: '-g',
                        overrides: [
                            'ubuntu-jammy': [
                                preBuildScript: '''
                                    echo "machine zextras.jfrog.io" >> auth.conf
                                    echo "login $USERNAME" >> auth.conf
                                    echo "password $SECRET" >> auth.conf
                                    mv auth.conf /etc/apt
                                    echo "deb [trusted=yes] https://zextras.jfrog.io/artifactory/ubuntu-devel jammy main" > zextras.list
                                    mv zextras.list /etc/apt/sources.list.d/
                                '''
                            ],
                            'ubuntu-noble': [
                                preBuildScript: '''
                                    echo "machine zextras.jfrog.io" >> auth.conf
                                    echo "login $USERNAME" >> auth.conf
                                    echo "password $SECRET" >> auth.conf
                                    mv auth.conf /etc/apt
                                    echo "deb [trusted=yes] https://zextras.jfrog.io/artifactory/ubuntu-devel noble main" > zextras.list
                                    mv zextras.list /etc/apt/sources.list.d/
                                '''
                            ],
                            'rocky-8': [
                                preBuildScript: '''
                                    echo "[Zextras]" > zextras.repo
                                    echo "baseurl=https://$USERNAME:$SECRET@zextras.jfrog.io/artifactory/centos8-devel/" >> zextras.repo
                                    echo "enabled=1" >> zextras.repo
                                    echo "gpgcheck=0" >> zextras.repo
                                    echo "gpgkey=https://$USERNAME:$SECRET@zextras.jfrog.io/artifactory/centos8-devel/repomd.xml.key" >> zextras.repo
                                    mv zextras.repo /etc/yum.repos.d/zextras.repo
                                '''
                            ],
                            'rocky-9': [
                                preBuildScript: '''
                                    echo "[Zextras]" > zextras.repo
                                    echo "baseurl=https://$USERNAME:$SECRET@zextras.jfrog.io/artifactory/rhel9-devel/" >> zextras.repo
                                    echo "enabled=1" >> zextras.repo
                                    echo "gpgcheck=0" >> zextras.repo
                                    echo "gpgkey=https://$USERNAME:$SECRET@zextras.jfrog.io/artifactory/rhel9-devel/repomd.xml.key" >> zextras.repo
                                    mv zextras.repo /etc/yum.repos.d/zextras.repo
                                '''
                            ],
                        ]
                    ])
                }
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
                uploadStage(
                    packages: yapHelper.resolvePackageNames()
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
