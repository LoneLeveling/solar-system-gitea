pipeline{
    agent any
    tools{
        nodejs 'nodes-22-6-0'
    }

     environment {
        JAVA_HOME = "/opt/homebrew/opt/openjdk@21"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
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

        stage('Unit Testing'){
            steps{
                sh 'npm test'
            }
        }
        }
    }
 }
}

