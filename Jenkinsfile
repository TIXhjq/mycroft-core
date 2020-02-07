pipeline {
    agent any
    options {
        // Running builds concurrently could cause a race condition with
        // building the Docker image.
        disableConcurrentBuilds()
    }
    stages {
        // Run the build in the against the dev branch to check for compile errors
        stage('Run Integration Tests') {
            when {
                anyOf {
                    branch 'testing/behave'
                    branch 'dev'
                    branch 'master'
                    changeRequest target: 'dev'
                }
            }
            steps {
                sh 'docker build --no-cache --target voigt_kampff -t mycroft-core:latest .'
                timeout(time: 60, unit: 'MINUTES')
                {
                    sh 'docker run \
                        -v "$HOME/voigtmycroft:/root/.mycroft" \
                        --device /dev/snd \
                        -e PULSE_SERVER=unix:${XDG_RUNTIME_DIR}/pulse/native \
                        -v ${XDG_RUNTIME_DIR}/pulse/native:${XDG_RUNTIME_DIR}/pulse/native \
                        -v ~/.config/pulse/cookie:/root/.config/pulse/cookie \
                        mycroft-core:latest'
                }
            }
        }
        
    }
    post {
        always('Important stuff') {
            sh 'mv $HOME/voigtmycroft/allure-result allure-result'
            script {
                allure([
                    includeProperties: false,
                    jdk: '',
                    properties: [],
                    reportBuildPolicy: 'ALWAYS',
                    results: [[path: 'allure-result']]
                ])
            }
            sh 'scp allure-port.zip root@157.245.127.234:~' + BRANCH_NAME
            sh(
                label: 'Docker container and image cleanup',
                script: '''
                    docker container prune --force
                    docker image prune --force
                '''.stripIndent()
            )
        }
    }
}
