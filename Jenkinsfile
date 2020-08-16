#!groovy


properties(
    [
        parameters(
            [
                string(description: 'CI Message', defaultValue: '{}', name: 'CI_MESSAGE')
            ]
        ),
        pipelineTriggers(
           [[$class: 'CIBuildTrigger',
               noSquash: true,
               providerData: [
                   $class: 'RabbitMQSubscriberProviderData',
                   name: env.FEDORA_CI_MESSAGE_PROVIDER,
                   overrides: [
                       topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
                       queue: 'osci-pipelines-queue-15'
                   ],
                   checks: [
                       [field: '$.artifact.release', expectedValue: '^f34$'],
                   ]
               ]
           ]]
        )
    ]
)

def msg
def artifactId
def additionalArtifactIds
def allTaskIds = [] as Set

pipeline {

    agent none

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

                                build job: 'fedora-ci/rpmdeplint-pipeline/master', wait: false, parameters: [ string(name: 'ARTIFACT_ID', value: artifactId), string(name: 'ADDITIONAL_ARTIFACT_IDS', value: additionalArtifactIds) ]
                            }
                        }
                    }
                }
            }
        }
    }
}
