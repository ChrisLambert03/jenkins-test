pipeline {
    agent any

    options {
        retry(1) // Retry once if any stage fails
        //catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') // Catch errors and set failure status
    }

    stages {
        
        stage('Setting Build Status on Github'){
            steps {
            setBuildStatus("Build ${BUILD_DISPLAY_NAME} Pending", "PENDING")
            }
        }

        stage('creating asset'){
            steps{
               sh' echo hello-world > release.md'
               sh 'echo hello-world > body.txt'
            }
        }
        stage('Making github release'){
            steps{
                createGitHubRelease(
                name: "Test Release",
                credentialId: 'Github-PAT',
                 repository: 'ChrisLambert03/jenkins-test',
                tag: "v1.0.${env.BUILD_NUMBER}",
                commitish: "${env.GIT_COMMIT}",
                bodyFile: 'release.md',
                draft: false,
                prerelease: false
                )
            }
        }
        stage("uploading assets"){
            steps{
                uploadGithubReleaseAsset(
                credentialId: 'Github-PAT',
                repository: 'ChrisLambert03/jenkins-test',
                tagName: "v1.0.${env.BUILD_NUMBER}", 
                uploadAssets: [
                [filePath: 'body.txt'], 
                    ]
                )
            }
        }
    }
    post{
        success{
            setBuildStatus("Build ${BUILD_DISPLAY_NAME} Complete", "SUCCESS")
        }
        failure{
            setBuildStatus("Build ${BUILD_DISPLAY_NAME} Failed", "FAILURE")
        }
    }
}

void setBuildStatus(String message, String state) {
    step([
        $class: "GitHubCommitStatusSetter",
        reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/ChrisLambert03/jenkins-test.git"],
        contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/build-test"],
        errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
        statusResultSource: [ $class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]] ]
    ])
}
