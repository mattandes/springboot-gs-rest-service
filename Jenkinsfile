#!groovy
// Artifcatory Server Id from Jenkins global config
def artifactoryServer = "artifactory"

// Artifactory Resolver Repo
def artifactoryResolverRepo = "maven-all"

// Artifactory snapshot repo to publish to
def artifactoryDeployRepo = "libs-snapshot-local"

// Artifactory release repo to promote to
def artifactoryPromoteRepo = "libs-release-local"

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
                    repo: artifactoryDeployRepo
                )
            }
        }
        stage('Build') {
            steps {
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
                rtAddInteractivePromotion (
                    serverId: artifactoryServer,
                    targetRepo: artifactoryPromoteRepo,
                    comment: 'Promoted via Jenkins',
                    status: 'Released',
                    sourceRepo: artifactoryDeployRepo,
                    failFast: true,
                    copy: false
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