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
    stages {
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
