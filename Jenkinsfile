#!groovy


def msg
def artifactId
def additionalArtifactIds
def allTaskIds = [] as Set

pipeline {

    agent {
        label 'master'                                                                                                                                        
    }

    // triggers {
    //    ciBuildTrigger(
    //        noSquash: true,
    //        providerList: [
    //            rabbitMQSubscriber(
    //                name: env.FEDORA_CI_MESSAGE_PROVIDER,
    //                overrides: [
    //                    topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
    //                    queue: 'osci-pipelines-queue-15'
    //                ],
    //                checks: [
    //                    [field: '$.artifact.release', expectedValue: '^f34$']
    //                ]
    //            )
    //        ]
    //    )
    // }

    parameters {
        string(name: 'CI_MESSAGE', defaultValue: '{}', description: 'CI Message')
    }

    stages {
        stage('Trigger Testing') {
            steps {
                script {
                    msg = readJSON text: CI_MESSAGE

                    if (msg) {
                        def fedUpdateId = msg['artifact']['id']
                        // the id in the messsage looks like this: FEDORA-2020-98a8f2073e-bf595b88d9f1e08f534a8b929a60cec255cc3952
                        // but the correct Fedora/Bodhi Id is just "FEDORA-2020-98a8f2073e"
                        fedUpdateId = fedUpdateId.reverse().split('-', 2)[1].reverse()

                        msg['artifact']['builds'].each { build ->
                            allTaskIds.add("koji-build:${build['task_id']}")
                        }

                        if (allTaskIds) {
                            // we can run rpmdeplint just once and then report on the whole update;
                            // that's why we are using the composite artifact here. example:
                            // (koji-build:52630695,koji-build:52626876)->fedora-update:FEDORA-2020-98a8f2073e
                            artifactId = "(${allTaskIds.join(',')})->fedora-update:${fedUpdateId}"

                            build job: 'fedora-ci/rpmdeplint-pipeline/master', wait: false, parameters: [ string(name: 'ARTIFACT_ID', value: artifactId) ]
                        }
                    }
                }
            }
        }
    }
}
