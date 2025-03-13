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

        stage('Dependencies Scanning'){
            parallel{
                stage('Dependencies Audit'){

                steps{
                    sh 'npm audit --audit-level=critical'
                }

                }

                stage('OWASP-Dependency Check'){
                    steps{
                        dependencyCheck additionalArguments: '''
                            --scan \'./\'
                            --out \'./\'
                            --format \'ALL\'
                            --prettyPrint''', odcInstallation: 'OWASP-DepCheck-12'

                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true
                    }
                }
            }

        }
    }
}