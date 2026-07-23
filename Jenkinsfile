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
            dependencyCheck additionalArguments: '''--scan \\\'./\\\'
                       --out \\\'./\\\'
                       --format \\\'ALL\\\'        
                      --prettyPrint''', odcInstallation: 'OWASP-DeCheck-12'

                      dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true
         }
        }
        }
    }
 }
}

