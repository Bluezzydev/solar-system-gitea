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
                        sh 'npm audit --audit-level=critical || true'
                    }
                }

                stage('OWASP') {
                    steps {
                        dependencyCheck additionalArguments: '''
                            --scan ./ 
                            --format ALL 
                            --disableNodeJS
                        ''',
                        odcInstallation: 'Dependency-Check'
                        dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                        publishHTML([reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'Security Report'])
                    }
                }
            }
        }

        stage('Unit Testing') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'mongo-db',
                        usernameVariable: 'MONGO_USER',
                        passwordVariable: 'MONGO_PASS'
                    )
                ]) {
                    script {
                        // Proper URI encoding using Groovy's URLEncoder
                        def encodedPass = URLEncoder.encode(env.MONGO_PASS, "UTF-8")
                        env.MONGO_URI = "mongodb+srv://${env.MONGO_USER}:${encodedPass}@supercluster.d83jj.mongodb.net/superData?retryWrites=true&w=majority&authMechanism=SCRAM-SHA-1"
                    }
                    sh '''
                        echo "Testing connection to MongoDB..."
                        npm test
                    '''
                }
            }
        }
    }
}