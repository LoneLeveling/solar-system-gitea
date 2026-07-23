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
         stage('NPM Dependency Audit'){
            sh '''
            npm audit --audit-level=critical
            echo $?
            '''
         }
    }
 }
}
