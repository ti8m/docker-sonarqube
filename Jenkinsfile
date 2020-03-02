#!groovy

pipeline {
    agent { label 'linux-tst-zone' }
    tools {
        jdk 'jdk-9_latest'
        maven 'Maven 3.x.x'
    }
    options {
        timestamps()
        timeout(time: 2, unit: 'HOURS')
        ansiColor('xterm')
        gitLabConnection('gitlab.ti8m.ch')
        buildDiscarder(logRotator(numToKeepStr: '25', artifactNumToKeepStr: '5'))
    }
    environment {
        registry = "gitlab.ti8m.ch:5043"
        registryCredential = 'bef49692-2225-4de5-9bb5-bead5daa6664'
        dockerTag = "${registry}/ti8m-forge-internal/sonarqube:7.9-community-ti8m"
    }
    triggers {
        gitlab(
                triggerOnPush: true,
                triggerOnMergeRequest: true,
                branchFilterType: 'All'
        )
    }
    stages {
        stage('Build image') {
            steps {
                sh(
                        script: """
                            cd 7/community
                            docker build -t ${dockerTag} .
                        """
                )
            }
        }
        stage('Push image') {
            steps {
                script {
                    docker.withRegistry("https://${registry}", "${registryCredential}") {
                        sh(
                                script: """
                                    docker push ${dockerTag}
                                """
                        )
                    }
                }
            }
        }
    }
    post {
        success {
            script {
                if (!hudson.model.Result.SUCCESS.equals(currentBuild.getPreviousBuild()?.getResult())) {
                    emailext(
                            subject: "GREEN: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                            body: '${JELLY_SCRIPT,template="html"}',
                            recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
//                            to: "foo@bar.ch",
//                            replyTo: "foo@bar.ch"
                    )
                }
            }
            updateGitlabCommitStatus(
                    name: 'build',
                    state: 'success'
            )
        }
        unstable {
            emailext(
                    subject: "YELLOW: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: '${JELLY_SCRIPT,template="html"}',
                    recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
//                    to: "foo@bar.ch",
//                    replyTo: "foo@bar.ch"
            )
            updateGitlabCommitStatus(
                    name: 'build',
                    state: 'failed'
            )
        }
        failure {
            emailext(
                    subject: "RED: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                    body: '${JELLY_SCRIPT,template="html"}',
                    recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
//                    to: "foo@bar.ch",
//                    replyTo: "foo@bar.ch"
            )
            updateGitlabCommitStatus(
                    name: 'build',
                    state: 'failed'
            )
        }
    }
}
