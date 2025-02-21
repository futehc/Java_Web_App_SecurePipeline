void setBuildStatus(String message, String state) {
    step([
        $class: "GitHubCommitStatusSetter",
        reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/masked/masked"],
        contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/build-status"],
        errorHandlers: [
            [$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]
        ],
        statusResultSource: [$class: "ConditionalStatusResultSource", results: [
            [$class: "AnyBuildResult", message: message, state: state]
        ]]
    ]);
}

def test_summary;

pipeline {
    agent any
    environment {
        DEPL = "masked"
        DEPL_NAMESPACE = "masked"
        IMAGE = "masked/masked"
        DOCKERHUB_CREDS = credentials('masked')
        ANSIBLE_SLACK_TOKEN = credentials('masked')
        AZURE_STORAGE_ACCOUNT_NAME = "masked"
        AZURE_STORAGE_CONTAINER_NAME = "masked"
    }
    stages {
        stage("OWASP Dependency Check") {
            steps {
                script {
                    try {
                        setBuildStatus("Running OWASP Dependency Check", "PENDING");
                        sh "dependency-check.sh --project masked --scan ."
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        setBuildStatus("OWASP Dependency Check failed.", "FAILURE");
                        throw e
                    }
                }
            }
        }
        stage("TruffleHog Secret Scanning") {
            steps {
                script {
                    try {
                        setBuildStatus("Running TruffleHog", "PENDING");
                        sh "trufflehog --json ."
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        setBuildStatus("TruffleHog Secret Scanning failed.", "FAILURE");
                        throw e
                    }
                }
            }
        }
        stage("Checkov Security Scan") {
            steps {
                script {
                    try {
                        setBuildStatus("Running Checkov Security Scan", "PENDING");
                        sh "checkov -d ."
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        setBuildStatus("Checkov Security Scan failed.", "FAILURE");
                        throw e
                    }
                }
            }
        }
        stage("Scan WebApp") {
            steps {
                script {
                    try {
                        setBuildStatus("Scanning app", "PENDING");
                        withSonarQubeEnv() {
                            sh "mvn --no-transfer-progress clean verify sonar:sonar -Dsonar.projectKey=masked -Dmaven.test.skip=true"
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        setBuildStatus("Scanning app failed.", "FAILURE");
                        throw e
                    }
                }
            }
        }
        stage("Build WebApp") {
            steps {
                script {
                    try {
                        setBuildStatus("Building app", "PENDING");
                        sh "mvn --no-transfer-progress package -Dmaven.test.skip=true"
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        setBuildStatus("Building app failed.", "FAILURE");
                        throw e
                    }
                }
            }
        }
        stage("Test WebApp") {
            steps {
                script {
                    try {
                        setBuildStatus("Testing web app.", "PENDING");
                        sh "mvn --no-transfer-progress test || :"
                        sh "pkill chrome"
                    } catch (Exception e) {
                        setBuildStatus("Web app testing failed.", "FAILURE");
                        currentBuild.result = 'UNSTABLE';
                        throw e;
                    }
                }
            }
            post {
                always {
                    script {
                        test_summary = junit healthScaleFactor: 25.0, keepLongStdio: true, testResults: '**/target/surefire-reports/TEST-*.xml', skipPublishingChecks: true
                    }
                }
            }
        }
        stage("Push artifact to Ansible server") {
            steps {
                script {
                    try {
                        setBuildStatus("Push artifact", "PENDING");
                        sshPublisher(publishers: [sshPublisherDesc(configName: 'masked', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'imageBuild/', remoteDirectorySDF: false, removePrefix: 'target/', sourceFiles: 'target/masked.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)]);
                    } catch (Exception e) {
                        setBuildStatus("Artifact push failed.", "FAILURE");
                        currentBuild.result = 'FAILURE';
                        throw e;
                    }
                }
            }
        }
    }
    post {
        always {
            sh "mvn clean"
        }
        success {
            script {
                setBuildStatus("CI successful", "SUCCESS");
            }
        }
        unstable {
            script {
                setBuildStatus("CI unstable, maybe tests failed", "SUCCESS");
            }
        }
        failure {
            script {
                setBuildStatus("CI failed", "FAILURE");
            }
        }
    }
}
