#!groovy
// Artifcatory Server Id from Jenkins global config
def artifactoryServer = "artifactory"

// Artifactory Resolver Repo
def artifactoryResolverRepo = "jcenter"

// Artifactory snapshot repo to publish to
def artifactorySnapshotRepo = "libs-snapshot-local"

// Artifactory release staging repo to publish to
def artifactoryStagingRepo = "libs-staging-local"

// Artifactory production release repo to promote to
def artifactoryReleaseRepo = "libs-release-local"

pipeline {
    agent any
    stages {
        stage('Artifactory Config') {
            steps {
                rtBuildInfo(
                    maxBuilds: 10,
                    deleteBuildArtifacts: true
                )
                rtGradleResolver(
                    id: 'artifactory-resolver',
                    serverId: artifactoryServer,
                    repo: artifactoryResolverRepo
                )
                rtGradleDeployer(
                    id: 'artifactory-deployer',
                    serverId: artifactoryServer,
                    repo: artifactorySnapshotRepo
                )
            }
        }
        stage('Build') {
            steps {
                // sh './gradlew clean build'
                rtGradleRun(
                    usesPlugin: true,
                    useWrapper: true,
                    tasks: 'clean build artifactoryPublish',
                    switches: '--no-daemon',
                    resolverId: "artifactory-resolver",
                    deployerId: "artifactory-deployer"
                )
                rtPublishBuildInfo(
                    serverId: artifactoryServer
                )
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