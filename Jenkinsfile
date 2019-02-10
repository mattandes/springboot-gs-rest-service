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
                // Update the deploy repo to the release repo amd remove snapshot from version property
                script {
                    nexusDeployRepo = nexusReleaseRepoUrl

                    // Read the properties file
                    def fileContent = readFile 'gradle.properties'
                    // Retrieve the current version
                    def currentVersion
                    def regex = '(?i)^version\\s*=.*'
                    fileContent.split(System.getProperty("line.separator")).each { line ->
                        if (line =~ regex) {
                            currentVersion = line
                        }
                    }
                    // Determine release version by droping '-SNAPSHOT' off the end
                    def releaseVersion = currentVersion.replaceAll('(?i)-SNAPSHOT$','')
                    // Replace the line in the file with the new version
                    def newFileContent = fileContent.replace(currentVersion, releaseVersion)
                    // Write the file back to disk
                    writeFile([file: propertiesFile, text: newFileContent])
                }
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
