#!/usr/bin/env groovy

library 'fg-shared-lib'

pipeline {
    agent any

    parameters {
        booleanParam(name: 'refreshOnly', defaultValue: false, description: 'Refresh Jenkinsfile and exit.')
        string(name: 'tag', defaultValue: 'latest', description: 'Set a docker tag')
    }
    options {
        disableConcurrentBuilds()
    }
    environment {
        DOCKER_REGISTRY_CREDENTIAL_ID = "$DOCKER_REGISTRY_HUB_CREDENTIAL_ID"
        DOCKER_USER = "$DOCKER_REGISTRY_HUB_USER"
        DOCKER_IMAGE = 'nginx-brotli'
        DOCKER_TAG = "${params.tag ?: '1.16.1}"
        DOCKER_FQIN = "${DOCKER_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}"
    }

    stages {
        stage('Refresh only') {
            when {
                expression { params.refreshOnly }
            }
            steps {
                echo "Jenkinsfile refreshed and pipeline finished."
            }
        }
        stage('Start Pipeline') {
            when {
                expression { !params.refreshOnly }
            }
            stages {
                stage('Create and push docker image') {
                    steps {
                        script { dockerLogin env.DOCKER_REGISTRY_CREDENTIAL_ID }
                        sh "docker build --rm -t ${env.DOCKER_IMAGE} ."
                        sh "docker tag ${env.DOCKER_IMAGE} ${env.DOCKER_FQIN}"
                        sh "docker push ${env.DOCKER_FQIN}"
                    }
                    post {
                        success {
                            echo 'Cleanup dangling images ...'
                            sh 'docker image prune -f --filter "dangling=true"'
                        }
                    }
                }
            }
            post {
                cleanup {
                    script { dockerLogout }
                }
            }
        }
    }
}
