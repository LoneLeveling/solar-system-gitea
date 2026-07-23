pipeline{
    agent any
    tools{
        nodejs 'nodes-22-6-0'
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
         }
        }
        }
    }
 }
}

