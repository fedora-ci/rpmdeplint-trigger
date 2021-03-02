#!groovy


def msg
def artifactId
def additionalArtifactIds
def taskId
def allTaskIds = [] as Set

pipeline {

    agent {
        label 'rpmdeplint-trigger'
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '45', artifactNumToKeepStr: '100'))
        skipDefaultCheckout()
    }

    triggers {
       ciBuildTrigger(
           noSquash: true,
           providerList: [
               rabbitMQSubscriber(
                   name: env.FEDORA_CI_MESSAGE_PROVIDER,
                   overrides: [
                       topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
                       queue: 'osci-pipelines-queue-15'
                   ],
                   checks: [
                       [field: '$.artifact.release', expectedValue: '^f35$']
                   ]
               )
           ]
       )
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
                            taskId = allTaskIds[0]
                            artifactId = "koji-build:${taskId}"
                            additionalArtifactIds = allTaskIds.findAll{ it != taskId }.collect{ "koji-build:${it}" }.join(',')
                            echo "Skipped: https://pagure.io/fesco/issue/2583"
                            // build job: 'fedora-ci/rpmdeplint-pipeline/master', wait: false, parameters: [ string(name: 'ARTIFACT_ID', value: artifactId), string(name: 'ADDITIONAL_ARTIFACT_IDS', value: additionalArtifactIds) ]
                        }
                    }
                }
            }
        }
    }
}
