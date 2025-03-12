pipeline{
    agent any

    tools{
        nodejs 'nodejs-23-9-0'
    }

    stages{
        stage('VM Node Version'){
            steps{
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}