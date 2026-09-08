pipeline {
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.11.5-alpine3.18'
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
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            agent any
            environment {
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                // Restaurar archivos del stash
                unstash(name: 'compiled-results')
                
                // === DEPURACIÃ“N: Ver quÃ© archivos existen ===
                sh 'echo "=== Contenido del workspace ==="'
                sh 'ls -la'
                sh 'echo "=== Contenido de sources/ ==="'
                sh 'ls -la sources/'
                sh 'echo "=== Verificando add2vals.py ==="'
                sh 'test -f sources/add2vals.py && echo "OK: add2vals.py existe" || echo "ERROR: add2vals.py NO existe"'
                
                // Ejecutar PyInstaller
                sh "docker run --rm -v ${VOLUME} -w /src ${IMAGE} 'ls -la /src'"
                sh "docker run --rm -v ${VOLUME} -w /src ${IMAGE} 'pyinstaller -F add2vals.py'"
            }
            post {
                success {
                    archiveArtifacts 'sources/dist/add2vals'
                    sh "docker run --rm -v ${VOLUME} -w /src ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}