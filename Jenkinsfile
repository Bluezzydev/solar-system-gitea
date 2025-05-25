pipeline {
    agent any

    tools {
        nodejs 'nodejs24.1.0'
    }

    stages {
        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install --no-audit'
            }
        }

        stage('Scan') {
            parallel {
                stage('Audit') {
                    steps {
                        echo 'Running npm audit...'
                        sh '''
                            npm audit --audit-level=critical
                            echo $?
                        '''
                    }
                }

                stage('OWASP') {
                    steps {
                        dependencyCheck additionalArguments: '''--scan './' \
                            --out './' \
                            --format ALL \
                            --prettyPrint''',
                            odcInstallation: 'Dependency-Check'
                            dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', unstableTotalCritical: 1
                    }
                }
            }
        }
    }
}
