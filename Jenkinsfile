pipeline{
    agent any
    tools{
        nodejs 'nodes-22-6-0'
    }

     environment {
        JAVA_HOME = "/opt/homebrew/opt/openjdk@21"
        PATH = "/usr/local/bin:${JAVA_HOME}/bin:${env.PATH}"
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
           --disableYarnAudit
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
                timeout(time: 60, unit: 'SECONDS') {
           withSonarQubeEnv('sonar-qube-server') {
            sh 'echo $SONAR_SCANNER_HOME'
            sh '''
             $SONAR_SCANNER_HOME/bin/sonar-scanner \
             -Dsonar.projectKey=Solar-System-Project \
             -Dsonar.sources=app.js \
             -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info \
                '''
            }
            waitForQualityGate abortPipeline: true
            }
        }
        }
        stage('Build Docker Image'){
            steps{
            // sh 'printenv'
            sh 'docker build -t loneleveling/solar-system:$GIT_COMMIT .'
        }
        }

        stage('Trivy Vulnerability Scanner'){
            steps{
             sh """
        docker run --rm \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -v "$WORKSPACE":/workspace \
        aquasec/trivy:0.72.0 \
        image \
        --ignorefile /workspace/.trivyignore \
        --severity LOW,MEDIUM,HIGH \
        --exit-code 0 \
        --quiet \
        --format json \
        --output /workspace/trivy-image-MEDIUM-results.json \
        loneleveling/solar-system:$GIT_COMMIT

        docker run --rm \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -v "$WORKSPACE":/workspace \
        aquasec/trivy:0.72.0 \
        image \
        --ignorefile /workspace/.trivyignore \
        --severity CRITICAL \
        --exit-code 1 \
        --quiet \
        --format json \
        --output /workspace/trivy-image-CRITICAL-results.json \
        loneleveling/solar-system:$GIT_COMMIT   
       """
        }
        post{
             always
               {
                // Humans don't like reading JSON so converting JSON to HTML Reports below :)

                 sh '''
                 docker run --rm \
                -v "$WORKSPACE":/workspace \
                aquasec/trivy:0.72.0 \
                convert \
                --format template \
                --template "@/contrib/html.tpl" \
                --output /workspace/trivy-image-MEDIUM-results.html \
                /workspace/trivy-image-MEDIUM-results.json      

                 docker run --rm \
                -v "$WORKSPACE":/workspace \
                aquasec/trivy:0.72.0 \
                convert \
                --format template \
                --template "@/contrib/html.tpl" \
                --output /workspace/trivy-image-CRITICAL-results.html \
                /workspace/trivy-image-CRITICAL-results.json

                 docker run --rm \
                 -v "$WORKSPACE":/workspace \
                 aquasec/trivy:0.72.0 \
                 convert \
                 --format template \
                 --template "@/contrib/junit.tpl" \
                 --output /workspace/trivy-image-MEDIUM-results.xml \
                /workspace/trivy-image-MEDIUM-results.json
                
                  docker run --rm \
                    -v "$WORKSPACE":/workspace \
                    aquasec/trivy:0.72.0 \
                    convert \
                    --format template \
                    --template "@/contrib/junit.tpl" \
                    --output /workspace/trivy-image-CRITICAL-results.xml \
                    /workspace/trivy-image-CRITICAL-results.json
                 '''
                }
            }
        }
           stage('Debug Docker') {
    steps {
        sh '''
        echo "PATH=$PATH"
        which docker
        docker --version
        '''
    }
}
        stage('Push Docker Image'){

        steps {
    withCredentials([usernamePassword(
        credentialsId: 'docker-hub-credentials',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASS'
    )]) {
        sh '''
        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
        docker push loneleveling/solar-system:$GIT_COMMIT
        docker logout
        '''
    }
}
}
}
           //Archiving Junit and publishing HTML reports always do post build stage.
    post{
        always{
        // Archiving the XML files
            junit allowEmptyResults: true, testResults: 'test-results.xml'
            junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'
            junit allowEmptyResults: true, testResults: 'trivy-image-MEDIUM-results.xml'
            junit allowEmptyResults: true, testResults: 'trivy-image-CRITICAL-results.xml'
        //Publishing the HTML Reports
            publishHTML([allowMissing: true,alwaysLinkToLastBuild: true,keepAll: true,reportDir: '.',reportFiles: 'trivy-image-MEDIUM-results.html',reportName: 'Trivy MEDIUM Report'])
            publishHTML([allowMissing: true,alwaysLinkToLastBuild: true,keepAll: true,reportDir: '.',reportFiles: 'trivy-image-CRITICAL-results.html',reportName: 'Trivy CRITICAL Report'])
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'index.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Covergae HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
