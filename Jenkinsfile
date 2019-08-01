import groovy.json.JsonSlurperClassic

def amqZipUrl
def amq_broker_version
def amq_broker_redhat_version
def build_url
def build_id

node ("messaging-ci-01.vm2") {
    stage ("Trigger image build") {
        build(
        job: 'amq-broker-73-container-image-build',
        parameters: [
            [ $class: 'StringParameterValue', name: 'AMQ_VERSION', value: '7.5' ],
            [ $class: 'StringParameterValue', name: 'BUILD_URL', value: build_url ],
            [ $class: 'StringParameterValue', name: 'amq_broker_version', value: amq_broker_version ],
            [ $class: 'StringParameterValue', name: 'amq_broker_redhat_version', value: amq_broker_redhat_version ]
        ],
        propagate: false
        )
    }
}
