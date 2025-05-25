pipeline {
    agent any

    tools {
        nodejs 'nodejs24.1.0'
    }
    environment {
  MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
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
                            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'dependency check jenkins HTML Report', reportTitles: ''])
                            junit allowEmptyResults: true, stdioRetention: 'ALL', testResults: 'dependency-check-junit.xml'
                            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'dependency check HTML Report', reportTitles: ''])
                    }
                }
            }
        }
        stage('unit testing') {
  steps {
    withCredentials([usernamePassword(credentialsId: 'mongo-db', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
   
        echo "Using MongoDB credentials: $MONGO_USERNAME"        
      echo 'Running unit tests...'
      sh 'npm test'
          }
    }
  }
}
        
    }

