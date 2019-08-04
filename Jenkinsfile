import groovy.json.JsonSlurperClassic

def builder = "AMQ7-Pipeline"
def amqZipUrl
def amq_broker_version
def amq_broker_redhat_version
def build_url
def build_id

node ("messaging-ci-01.vm2") {
    stage('prepare amq master prod branch') {
        build(
        job: 'update_master_branch',
        propagate: false
        )
    }
    stage('prepare prod branch') {
        build(
        job: 'amq-prepare-pnc-branch',
        parameters: [
            [ $class: 'StringParameterValue', name: 'product_branch', value: 'pre-7.5.x' ],
            [ $class: 'StringParameterValue', name: 'rebase_branch', value: 'master' ]
        ],
        propagate: false
        )
    }
    stage('build amq 7.5') {
        def amq = build(
        job: 'amq-pnc-build',
        parameters: [
            [ $class: 'StringParameterValue', name: 'BUILDCONFIG', value: '7.5' ],
            [ $class: 'StringParameterValue', name: 'TEMPBUILD', value: 'true' ]
        ],
        propagate: false
        )
        if (amq.result != 'SUCCESS') {
          def emailBody = """
            Building of AMQ failed.

            See job for details: ${amq.absoluteUrl}
          """.stripIndent().trim()
          node {
            emailext body: emailBody, subject: "AMQ Broker nightly prod build ${new Date().format('yyyy-MM-dd')}", to: 'hgao@redhat.com'
            throw new Exception("Production job failed. Cannot continue.")
          }
        }
        sh "echo running"
        def amqVariables = amq.getBuildVariables();
        build_url = "${amqVariables.BUILD_URL}"
        sh "echo $build_url"
        build_id = "${amqVariables.BUILD_ID}"
        sh "rm -f repository-artifact-list.txt"
        sh "wget ${amq.absoluteUrl}/artifact/amq-broker-7.5.0.ER1/extras/repository-artifact-list.txt"
        amq_broker_redhat_version = sh(script: "grep org.jboss.rh-messaging.amq:amq-broker: repository-artifact-list.txt|cut -d':' -f3", returnStdout: true)
        sh "echo amq_broker_redhat_version $amq_broker_redhat_version"
        amq_broker_version = amq_broker_redhat_version.substring(0, amq_broker_redhat_version.indexOf('-'))
        sh "echo amq_broker_version amq_broker_version"
    }
    stage ("Update Stagger") {
        echo "====== Updating stagger"
        checkout scm
        sh "sh ./scripts/pushamq.sh $build_id $build_url $amq_broker_version $amq_broker_redhat_version"
    }
    stage ("Send Email") {
        sh "echo ====== Sending mail"
        build(
        job: 'sendSuccessEmail',
        parameters: [
            [ $class: 'StringParameterValue', name: 'AMQ_VERSION', value: '7.5' ],
            [ $class: 'StringParameterValue', name: 'BUILD_URL', value: build_url ],
            [ $class: 'StringParameterValue', name: 'amq_broker_version', value: amq_broker_version ],
            [ $class: 'StringParameterValue', name: 'amq_broker_redhat_version', value: amq_broker_redhat_version ]
        ],
        propagate: false
        )

    }
    stage ("Start image build") {
        sh "echo ====== Buidling image" 
        build(
        job: 'amq-broker-73-container-image-build',
        parameters: [
            [ $class: 'StringParameterValue', name: 'builder', value: builder ],
            [ $class: 'StringParameterValue', name: 'AMQ_VERSION', value: '7.5' ],
            [ $class: 'StringParameterValue', name: 'BUILD_URL', value: build_url ],
            [ $class: 'StringParameterValue', name: 'amq_broker_version', value: amq_broker_version ],
            [ $class: 'StringParameterValue', name: 'amq_broker_redhat_version', value: amq_broker_redhat_version ]
        ],
        propagate: false
        )
    }


}
