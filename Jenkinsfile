// Build daily on master
String CRON_STRING = BRANCH_NAME == "master" ? "@daily" : ""

pipeline {

    agent {
        label 'BLDLNX'
    }
    options {
        // checkoutToSubdirectory(SRC_DIR)
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '2'))
        ansiColor('xterm')
        timestamps()
    }
    environment {
        PIP_DISABLE_PIP_VERSION_CHECK = 1
        PIP_INDEX_URL = 'https://artprod.dev.bloomberg.com/artifactory/api/pypi/bloomberg-pypi/simple'
        PYEXEC = 'python3.7'
        VENV_PATH = "$(pwd)/test-venv"
        PATH = "$VENV_PATH/bin:$PATH"
    }
    triggers {
        cron(CRON_STRING)
    }
    stages {
        stage('Setup') {
            steps {
                sh "python3.7 -m venv test-venv"
                sh "$(VENV_PATH)/bin/pip install --progress-bar=off -U -r tests/requirements-tools.txt -r tests/requirements-libraries.txt"
            }
        }
        stage('Run Linter') {
            steps {
                script {
                    sh "$VENV_PATH/bin/flake8-diff -v -v -v master"
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    sh "$VENV_PATH/bin/pytest -n8 -vv tests"
                }
            }
        }
    }
    post {
        success {
            deleteDir()
        }
        failure {
            junit "src/.tox/*_integration.xml"
            emailext subject: "${env.JOB_NAME} #${env.BUILD_NUMBER} is broken",
                     to: "lkisskollar@bloomberg.net, mcorcherojim@bloomberg.net",
                     body: '''Jenkins job: ${PROJECT_URL}
Build details: ${BUILD_URL}
Log excerpt:
${BUILD_LOG}
'''
        }
    }
}