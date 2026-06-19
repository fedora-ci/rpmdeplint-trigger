#!groovy


def msg
def allTaskIds = [] as Set

pipeline {

    agent none

    options {
        buildDiscarder(logRotator(daysToKeepStr: '3', artifactNumToKeepStr: '100'))
        skipDefaultCheckout()
    }

    triggers {
       ciBuildTrigger(
           noSquash: true,
           providerList: [
               rabbitMQSubscriber(
                   name: 'RabbitMQ',
                   overrides: [
                       topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
                       queue: 'osci-pipelines-queue-15'
                   ],
                   checks: [
                       [field: '$.update.release.dist_tag', expectedValue: '^f45$']
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

                        build job: 'fedora-ci/rpmdeplint-pipeline/master', wait: false, parameters: [
                            string(name: 'BODHI_UPDATE_ID', value: msg['update']['updateid']),
                            string(name: 'ARTIFACT_IDS', value: allTaskIds.collect{ "koji-build:${it}" }.join(',')),
                            string(name: 'DIST_GIT_BRANCH', value: msg['update']['release']['branch']),
                        ]
                    }
                }
            }
        }
    }
}
