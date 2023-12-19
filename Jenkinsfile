pipeline {
    agent none 
    stages {
        stage('Lint') {
            agent {
                docker {
                    image 'cytopia/pylint'
                }
            }
            steps {
                sh 'pylint sources/*.py'
            }
        }
        
        stage('Build') { 
            agent {
                docker {
                    image 'python:3.12.1-alpine3.19' 
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py' 
                stash(name: 'compiled-results', includes: 'sources/*.py*') 
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }

        stage('Generate Documentation') {
            agent {
                docker {
                    image 'sphinxdoc/sphinx' 
                }
            }
            steps {
                sh '''
                sphinx-quickstart -q -p "My Project" -a "Author Name" docs
                sphinx-build -M html docs docs/build/
                '''
                archiveArtifacts artifacts: 'docs/build/**/*.html', allowEmptyArchive: true
            }
        }

        stage('Deliver') {
            agent any
            environment {
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}
