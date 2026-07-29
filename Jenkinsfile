pipeline{
    agent any
    tools{
        nodejs 'nodes-22-6-0'
    }

     environment {
        JAVA_HOME = "/opt/homebrew/opt/openjdk@21"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
        MONGO_URI = "mongodb+srv://cluster0.7mmrff9.mongodb.net/superData?retryWrites=true&w=majority&appName=Cluster0"
        // MONGO_DB_CREDS=credentials('mongo-db-credentials')
        MONGO_USERNAME=credentials('mongo-db-username')
        MONGO_PASSWORD=credentials('mongo-db-password')
        SONAR_SCANNER_HOME=tool('sonarqube-scanner-801')
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

         }
         }
        }
    }
         stage('Unit Testing'){
            // options{ retry(2) }
            steps{
            sh '''
            echo $MONGO_DB_CREDS
            echo $MONGO_DB_CREDS_USR
            echo $MONGO_DB_CREDS_PSW
            '''
            //withCredentials([usernamePassword(credentialsId: 'mongo-db-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {
            sh 'npm test'
            // }
            }
        }

        stage('Code Coverage'){
     steps{
             //withCredentials([usernamePassword(credentialsId: 'mongo-db-credentials', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]){
                catchError(buildResult: 'SUCCESS', message: 'OOPS!! Build failed but continuing further', stageResult: 'UNSTABLE') {
                sh 'npm run coverage'
            }
        }
        }
        stage('SAST - SonarQube'){
            steps{
                sh 'echo $SONAR_SCANNER_HOME'
                sh '''
             $SONAR_SCANNER_HOME/bin/sonar-scanner \
             -Dsonar.projectKey=Solar-System-Project \
             -Dsonar.sources=. \
             -Dsonar.host.url=http://localhost:9000 \
             -Dsonar.login=sqp_c58a0063bedfef6f8612c49d869b817e98aee8c5
                '''
            }
        }
        }

        //Archiving Junit and publishing HTML reports always do post build stage.
    post{
    always{
            junit allowEmptyResults: true, testResults: 'test-results.xml'
            junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'index.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Covergae HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
 