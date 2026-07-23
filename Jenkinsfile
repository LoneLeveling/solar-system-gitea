pipeline{
    agent any
    stages{
        stage('VM Node Version'){
            steps{
            sh 'export PATH="/Users/abhisheksharma/.nvm/versions/node/v22.23.1/bin:$PATH"'
            sh 'node -v' 
            sh 'npm -v'
        }
    }
 }
}
