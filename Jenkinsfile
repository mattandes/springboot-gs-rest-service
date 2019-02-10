#!groovy
// Nexus Server URL using NEXUS_URL environment variable set by Jenkins config
def nexusBaseURL = "${env.NEXUS_URL}/repository"

// Nexus resolver repo URL
def nexusResolverUrl = "${nexusBaseURL}/maven-public/"

// Nexus snapshot repo URL to publish to
def nexusSnapshotRepoUrl = "${nexusBaseURL}/maven-snapshots/"

// Nexus release repo URL to publish to
def nexusReleaseRepoUrl = "${nexusBaseURL}/maven-releases/"

// Set the default repo to deploy to
def nexusDeployRepo = nexusSnapshotRepoUrl

pipeline {
    agent any
    parameters {
        booleanParam(name: 'RELEASE_BUILD', defaultValue: false, description: 'Determine whether this is a release build or not. Release builds will increment artifact version numbers.')
    }
    stages {
        stage('Switch to Release Version') {
            when {
                expression { return params.RELEASE_BUILD }
            }
            steps {
                // Update the deploy repo to the release repo
                script {
                    nexusDeployRepo = nexusReleaseRepoUrl
                }
                sh '''
                    CURRENT_VERSION=`cat gradle.properties | grep -e "^version="`
                    NEW_VERSION=`echo ${CURRENT_VERSION} | sed "s/-SNAPSHOT\$//"`
                    sed -i "s/${CURRENT_VERSION}/${NEW_VERSION}/" gradle.properties
                '''
            }
        }
        stage('Build') {
            steps {
                sh "./gradlew --no-daemon -PnexusResolveUrl='${nexusResolverUrl}' -PnexusDeployUrl='${nexusDeployRepo}' clean build upload"
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'build/libs/**/*.jar', fingerprint: true
            junit testResults: 'build/test-results/**/*.xml'
        }
    }
}
