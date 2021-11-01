pipeline {
    agent {
        label "master"
    }

    environment {
        GIT_COMMIT_SHORT = sh(
            script: "printf \$(git rev-parse --short ${GIT_COMMIT})",
            returnStdout: true
        )
        GIT_COMMIT_DATE = sh(
            script: "printf \$(git show -s --format=%ci ${GIT_COMMIT})",
            returnStdout: true
        )

        ARTIFACT_REPOSITORY = "https://index.docker.io/v1"
        CREDENTIAL_ID = "docker-hub"

        BUILD_IMAGE = "fe-angular:${GIT_COMMIT_DATE.take(10)}_${GIT_COMMIT_SHORT}"
        DEPLOY_IMAGE = "tranhoangcore/${BUILD_IMAGE}"
    }

    stages {
        stage ("Validate") {
            steps {
                sh "docker version"
                sh "node --version"
                sh "npm version"
            }
        }

        stage ("Build docker image fe-angular") {
            steps {
                sh "docker build --pull --no-cache -f Dockerfile -t ${BUILD_IMAGE} ."
            }
        }

        stage ("Push docker image fe-angular to hub") {
            steps {
                withDockerRegistry([credentialsId: "${CREDENTIAL_ID}", url: ""]) {
                    sh "docker tag ${BUILD_IMAGE} ${DEPLOY_IMAGE}"
                    sh "docker push ${DEPLOY_IMAGE}"
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace'
            sh "docker rmi ${DEPLOY_IMAGE} || true"
            sh "docker rmi ${BUILD_IMAGE} || true"
            deleteDir() 
        }
    }
}