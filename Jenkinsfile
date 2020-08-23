#!groovy


def msg
def artifactId
def additionalArtifactIds
def allTaskIds = [] as Set

pipeline {

    agent {
        label 'master'                                                                                                                                        
    }

    parameters {
        string(name: 'CI_MESSAGE', defaultValue: '{}', description: 'CI Message')
    }

    stages {
        stage('Trigger Testing') {
            steps {
                script {
                    msg = readJSON text: CI_MESSAGE

                    if (msg) {
                        msg['artifact']['builds'].each { build ->
                            allTaskIds.add(build['task_id'])
                        }

                        if (allTaskIds) {
                            allTaskIds.each { taskId ->
                                artifactId = "koji-build:${taskId}"
                                additionalArtifactIds = allTaskIds.findAll{ it != artifactId }.collect{ "koji-build:${it}" }.join(',')

                                // build job: 'fedora-ci/rpmdeplint-pipeline/master', wait: false, parameters: [ string(name: 'ARTIFACT_ID', value: artifactId), string(name: 'ADDITIONAL_ARTIFACT_IDS', value: additionalArtifactIds) ]
                            }
                        }
                    }
                }
            }
        }
    }
}
