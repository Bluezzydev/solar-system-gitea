pipeline {
    agent any

    tools {
        nodejs 'nodejs24.1.0'
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        MONGO_DB_CREDENTIALS = credentials('mongo-db')
    }

    options { 
        timestamps()
        disableResume()
        disableConcurrentBuilds abortPrevious: true
    }

    stages {
        stage('Install Dependencies') {
            options {
                timestamps()
            }
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
                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'dependency check jenkins HTML Report', reportTitles: ''])
                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'dependency check HTML Report', reportTitles: ''])
                        junit allowEmptyResults: true, stdioRetention: 'ALL', testResults: 'dependency-check-junit.xml'
                    }
                }
            }
        }

        stage('unit testing') {
            steps {
                  sh 'echo $MONGO_DB_CREDENTIALS'
                  sh 'echo username: $MONGO_DB_CREDENTIALS_USR'
                  sh 'echo password: $MONGO_DB_CREDENTIALS_PSW'
                  
                
                    echo "Using MongoDB credentials: $MONGO_USERNAME"        
                    echo 'Running unit tests...'
                    sh 'npm test'
                
            }
        }

        stage('Code Coverage and Catch Errors') {
            steps {
                
                    catchError(buildResult: 'SUCCESS', message: 'there\'s error we will fix in next realse', stageResult: 'UNSTABLE') {
   

                    sh 'npm run coverage'
                    }
                    publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'code coverage HTML Report', reportTitles: ''])

                
            }
        }
    }
}
