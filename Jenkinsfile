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
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    
                    // --- COMANDOS DE DEPURACIÃ“N ---
                    sh 'echo "=== 1. Directorio actual ==="'
                    sh 'pwd'
                    sh 'echo "=== 2. Contenido del directorio actual ==="'
                    sh 'ls -la'
                    sh 'echo "=== 3. Contenido de la carpeta sources ==="'
                    sh 'ls -la sources || echo "ERROR: La carpeta sources NO existe"'
                    sh 'echo "=== 4. Lo que ve Docker dentro de /src ==="'
                    sh "docker run --rm -v ${VOLUME} -w /src ${IMAGE} 'ls -la /src'"
                    // --------------------------------
                    
                    sh "docker run --rm -v ${VOLUME} -w /src ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v ${VOLUME} -w /src ${IMAGE} 'rm -rf build dist'"
                }
            }
        }
    }
}