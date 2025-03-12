pipeline{
    agent any

    tools{
        nodejs 'nodejs-23-9-0'
    }

    stages{
        stage('Installing Dependencies'){
            steps{
                sh 'npm install --no-audit'
            }
        }
    }
}