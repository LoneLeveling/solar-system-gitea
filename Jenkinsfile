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
            // sh 'docker build -t loneleveling/solar-system:$GIT_COMMIT .'
            // sh 'docker build -t brawd375/solar-system:$GIT_COMMIT .'
            sh 'docker build --platform linux/amd64 -t brawd375/solar-system:$GIT_COMMIT .'
            sh 'docker image ls'
            sh 'docker image inspect brawd375/solar-system:$GIT_COMMIT'
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
        brawd375/solar-system:$GIT_COMMIT

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
        brawd375/solar-system:$GIT_COMMIT   
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
    //     stage('Push Docker Image'){
    //         steps{
    //         withEnv(['PATH+DOCKER=/usr/local/bin']){
    //         withDockerRegistry(credentialsId: 'docker-hub-credentials', url: "") {
    //         sh 'docker push brawd375/solar-system:$GIT_COMMIT'
    //     }
    //     }
    //     }
    //   }

stage('Push Docker Image') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'docker-hub-credentials',
                usernameVariable: 'DOCKER_USERNAME',
                passwordVariable: 'DOCKER_PASSWORD'
            )
        ]) {
            sh '''
                printf '%s' "$DOCKER_PASSWORD" | /usr/local/bin/docker login \
                    --username "$DOCKER_USERNAME" \
                    --password-stdin

                /usr/local/bin/docker push brawd375/solar-system:$GIT_COMMIT

                /usr/local/bin/docker logout
            '''
        }
    }
}

stage('Deploy - AWS EC2') {
    when {
        branch 'feature/*'
    }
    steps {
        sshagent(credentials: ['was-dev-deploy-ec2-instance']) {
            sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@3.88.109.244  << EOF

                if sudo docker ps -a | grep -q 'solar-system'; then
                    echo 'Container found. Stopping...'
                    sudo docker stop solar-system
                    sudo docker rm solar-system
                    echo 'Container stopped and removed.'
                fi

                sudo docker run --name solar-system \\
                    -e MONGO_URI="$MONGO_URI" \\
                    -e MONGO_USERNAME="$MONGO_USERNAME" \\
                    -e MONGO_PASSWORD="$MONGO_PASSWORD" \\
                    -p 3000:3000 \\
                    -d brawd375/solar-system:$GIT_COMMIT

EOF
            '''
        }
    }
}

stage('Integration Testing - AWS EC2') {
    when {
        branch 'feature/*'
    }
    steps {
        sh 'printenv | grep -i branch'
        withAWS(credentials: 'aws-ec2-s3-lambda-creds', region: 'us-east-1b') {
            sh '''
                bash /Users/abhisheksharma/Desktop/solar-system-gitea/integration-testing-ec2.sh
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

