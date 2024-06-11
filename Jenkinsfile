pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.12-alpine'
                }
            }
            steps {
                sh 'python3.12 -m py_compile sources/prog.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'grihabor/pytest'
                }
            }
            steps {
                sh 'pytest -v --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit "test-reports/results.xml"
                }
            }
        }
        stage('Deliver') {
            agent any
            environment {
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux'
            }
            steps {
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F prog.py'"
                }
            }
            /*post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/prog"
                    sh "sudo chown -R jenkins:jenkins ${env.BUILD_ID}/sources/build"
                    sh "sudo chown -R jenkins:jenkins ${env.BUILD_ID}/sources/dist"
                    sh "rm -rf ${env.BUILD_ID}/sources/build ${env.BUILD_ID}/sources/dist"
                }
            }*/
        }
    } 
    post {
        always {
            echo 'Testing Jenkins build'
        }
        success {
            echo 'Jenkins build Done Successfully'
        }
        failure {
            echo 'Jenkins build Done with failure'
        }
    }
}
