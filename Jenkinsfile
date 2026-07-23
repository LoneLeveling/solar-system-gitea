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

stage('Check Environment') {
    steps {
        sh '''
            echo "PATH=$PATH"
            echo "JAVA_HOME=$JAVA_HOME"
            ls -l /opt/homebrew/opt/openjdk@21/bin/java
        '''
    }
}
stage('Check Java') {
    steps {
        sh '''
            which java
            java --version
            echo $JAVA_HOME
            /usr/libexec/java_home -V || true
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

