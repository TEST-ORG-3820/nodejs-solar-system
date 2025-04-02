pipeline{
    agent any

    tools{
        nodejs 'NODE-JS-23-9'
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
                stage('OWASP-Dependency Check') {
                    steps {
                       
                            dependencyCheck additionalArguments: """
                                --scan './'
                                --out './'
                                --format 'ALL'
                                --prettyPrint
                            """, odcInstallation: 'OWASP-Depcheck-12'


                            dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true
                            
                            junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'
    
                            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-jenkins.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        
                    }
                }


               
            }

        }
    }
}
