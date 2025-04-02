pipeline {
    agent any

    tools {
        nodejs 'NODE-JS-23-9'
    }

    environment {
    MONGO_URI = 'mongodb+srv://bala74573:uo6a6FqZxG6PSy4Q@cluster01.0xf4mlq.mongodb.net/'
    MONGO_USERNAME = 'bala74573'
    MONGO_PASSWORD = 'uo6a6FqZxG6PSy4Q'
    }


    stages {
        stage('Installing Dependencies') {
            steps {
                sh 'npm install --no-audit'
            }
        }

        stage('Dependencies Scanning') {
            parallel {
                stage('Dependencies Audit') {
                    steps {
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

                        dependencyCheckPublisher(
                            failedTotalCritical: 1,
                            pattern: 'dependency-check-report.xml',
                            stopBuild: true
                        )

                        junit allowEmptyResults: true, testResults: 'dependency-check-junit.xml'

                        publishHTML([
                            allowMissing: true,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: './',
                            reportFiles: 'dependency-check-jenkins.html',
                            reportName: 'Dependency Check HTML Report',
                            reportTitles: '',
                            useWrapperFileDirectly: true
                        ])
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t balakumarpalanisamy/solar-system:$GIT_COMMIT .'
            }
        }

        stage('Trivy Vulnerability Scanner') {
            steps {
                sh '''
                    trivy image balakumarpalanisamy/solar-system:$GIT_COMMIT \
                        --severity LOW,MEDIUM,HIGH \
                        --exit-code 0 \
                        --quiet \
                        --format json -o trivy-image-MEDIUM-results.json

                    trivy image balakumarpalanisamy/solar-system:$GIT_COMMIT \
                        --severity CRITICAL \
                        --exit-code 1 \
                        --quiet \
                        --format json -o trivy-image-CRITICAL-results.json
                '''
            }

            post {
                always {
                    sh '''
                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                            --output trivy-image-MEDIUM-results.html trivy-image-MEDIUM-results.json

                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/html.tpl" \
                            --output trivy-image-CRITICAL-results.html trivy-image-CRITICAL-results.json

                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                            --output trivy-image-MEDIUM-results.xml trivy-image-MEDIUM-results.json

                        trivy convert \
                            --format template --template "@/usr/local/share/trivy/templates/junit.tpl" \
                            --output trivy-image-CRITICAL-results.xml trivy-image-CRITICAL-results.json
                    '''
                }
            }
        }

        stage('Push Docker Image'){
            steps{
                withDockerRegistry(credentialsId: 'docker-hub-credentials', url: ""){ 
                    sh 'docker push balakumarpalanisamy/solar-system:$GIT_COMMIT'
                } 
            }
        }

        stage ('Deploy - Azure VM' ) {
            when {
                branch 'feature/*'
            }
            steps{
                script {
                    sshagent (['azure-vm']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no azureuser@4.206.91.54
                            if sudo docker ps -a | grep -q "solar-system"; then
                                echo "Container found. Stopping..."
                                sudo docker stop "solar-system" && sudo docker rm "solar-system"
                                echo "Container stopped and removed."
                            fi
                            sudo docker run --name solar-system \
                                -e MONGO_URI=$MONGO_URI \
                                -e MONGO_USERNAME=$MONGO_USERNAME \
                                -e MONGO_PASSWORD=$MONGO_PASSWORD \
                                -p 3000:3000 -d balakumarpalanisamy/solar-system:$GIT_COMMIT
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: './',
                reportFiles: 'trivy-image-CRITICAL-results.html',
                reportName: 'Trivy Image Critical Vul Report',
                reportTitles: '',
                useWrapperFileDirectly: true
            ])

            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: './',
                reportFiles: 'trivy-image-MEDIUM-results.html',
                reportName: 'Trivy Image Medium Vul Report',
                reportTitles: '',
                useWrapperFileDirectly: true
            ])
        }
    }
}

