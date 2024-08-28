def bobVersion = '0.0.5'
def bob = "docker run --rm " +
        '--env APP_PATH="`pwd`" ' +
        '--env RELEASE=${RELEASE} ' +
        "-v \"`pwd`:`pwd`\" " +
        "-v /var/run/docker.sock:/var/run/docker.sock " +
        "armdocker.rnd.ericsson.se/proj-orchestration-so/bob:${bobVersion}"

pipeline {
    agent {
        node {
            label 'so_slave'
        }
    }
    stages {
        stage('Clean') {
            steps {
                sh "${bob} clean"
                // sh 'git clean -xdff'
            }
        }
        stage('Submodule sync') {
            steps {
                sh 'git submodule sync'
                sh 'git submodule update --init --recursive'
                // Ensure that Bob has all of its dependencies 
                sh "${bob} --selftest"
            }
        }
        stage('Lint') {
            steps {
                sh "${bob} lint"
            }
        }
        stage('Build image and chart') {
            steps {
                sh "${bob} image"
            }
        }
        stage('Push image and chart') {
            when {
                expression { params.RELEASE == "true" }
            }
            steps {
                sh "${bob} push"
            }
        }
        stage('Generate input for DIT staging'){
            when {
                expression { params.RELEASE == "true" }
            }
            steps{
                sh "${bob} generate-output-parameters"
                archiveArtifacts 'artifact.properties'
            }
        }
    }

}
