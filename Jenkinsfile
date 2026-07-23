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
 }
}
