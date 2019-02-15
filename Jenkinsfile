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
                    CURRENT_VERSION_LINE=`cat gradle.properties | grep -e "^version="`
                    NEW_VERSION=`echo ${CURRENT_VERSION_LINE} | sed "s/-SNAPSHOT\$//"`
                    sed -i "s/${CURRENT_VERSION_LINE}/${NEW_VERSION}/" gradle.properties
                '''
            }
        }
        stage('Build') {
            steps {
                sh "./gradlew --no-daemon -PnexusResolveUrl='${nexusResolverUrl}' -PnexusDeployUrl='${nexusDeployRepo}' clean build upload"
            }
        }
        stage('Increment Version Number') {
            when {
                expression { return params.RELEASE_BUILD }
            }
            steps {
                sshagent(credentials: ['gitlab-root-private-key']) {
                    sh '''
                        git config user.name "Jenkins"
                        git config user.email "Jenkins@example.org"
                        git add gradle.properties
                        git commit -m "[Jenkins] Creating release version."
                        CURRENT_VERSION_LINE=`cat gradle.properties | grep -e "^version="`
                        CURRENT_VERSION=`echo ${CURRENT_VERSION_LINE} | cut -d'=' -f2`
                        git tag ${CURRENT_VERSION} -m "Release of version ${CURRENT_VERSION}."
                        NEW_VERSION=`./increment_version.sh -p ${CURRENT_VERSION}`
                        NEW_VERSION_LINE="version=${NEW_VERSION}-SNAPSHOT"
                        sed -i "s/${CURRENT_VERSION_LINE}/${NEW_VERSION_LINE}/" gradle.properties
                        git add gradle.properties
                        git commit -m "[Jenkins] Incrementing version."
                        git push --all
                        git push --tags
                    '''
                }
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
