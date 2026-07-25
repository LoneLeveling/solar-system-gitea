pipeline{
    agent any
    tools{
        nodejs 'nodes-22-6-0'
    }

     environment {
        JAVA_HOME = "/opt/homebrew/opt/openjdk@21"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
        MONGO_URI = "mongodb+srv://cluster0.7mmrff9.mongodb.net/superData?retryWrites=true&w=majority&appName=Cluster0"
        MONGO_DB_CREDS=credentials('mongo-db-credentials')
        }

            options {
    disableConcurrentBuilds()
         } 

    stages{
       
        stage('Installing Dependencies'){
            steps{
            sh 'npm install --no-audit'
         }
        }

        stage('Dependency Scanning'){
            parallel{
         stage('NPM Dependency Audit'){
            steps{
            sh '''
            npm audit --audit-level=critical
            echo $?
            '''
         }
         }
            
         stage('OWASP Dependency Check'){
            steps{
           dependencyCheck(
           odcInstallation: 'OWASP-DeCheck-12',
           additionalArguments: '''
           --scan .
           --out .
           --format ALL
           --prettyPrint
           '''
          )

          dependencyCheckPublisher(
          pattern: '**/dependency-check-report.xml',
          failedTotalCritical: 1,
          stopBuild: true
           )
            junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'

           publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'index.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
         }
        }
        }
        }
         stage('Unit Testing'){
            sh 'echo $MONGO_DB_CREDS'
            sh 'echo $MONGO_DB_CREDS_USR'
            sh 'echo $MONGO_DB_CREDS_PSW'
            // options{ retry(2) }
            steps{
            // withCredentials([usernamePassword(credentialsId: 'mongo-db-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
                sh 'npm test'
            // }
                        junit allowEmptyResults: true, testResults: 'test-results.xml'
            }
        }

        stage('Code Coverage'){
        steps{
            // withCredentials([usernamePassword(credentialsId: 'mongo-db-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]){
            catchError(buildResult: 'SUCCESS', message: 'OOPS!! Build failed but continuing further', stageResult: 'UNSTABLE') {
                sh 'npm run coverage'
            }
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Covergae HTML Report', reportTitles: '', useWrapperFileDirectly: true])
         }
        }
        }
}
 