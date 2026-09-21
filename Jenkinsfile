pipeline {

    agent any

    environment {
        COLLECTION = 'Collection Rapports.postman_collection.json'
        REPORT_DIR = 'reports/newman'
        REPORT_FILE = 'reports/newman/report.html'
        GIT_CREDENTIALS = 'github-inasse-ssh'
    }

    stages {

        stage('Installation Newman') {
            steps {
                powershell '''
                    npm install -g newman
                    npm install -g newman-reporter-htmlextra
                '''
            }
        }

        stage('Préparation du rapport') {
            steps {
                powershell '''
                    New-Item -ItemType Directory -Path ".\\reports\\newman" -Force
                '''
            }
        }

        stage('Exécution des tests Postman') {
            steps {
                powershell '''
                    newman run ".\\Collection Rapports.postman_collection.json" `
                        --reporters cli,htmlextra `
                        --reporter-htmlextra-export ".\\reports\\newman\\report.html"
                '''
            }
        }

        stage('Publication du rapport Jenkins') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports/newman',
                    reportFiles: 'report.html',
                    reportName: 'Rapport Newman',
                    reportTitles: 'Tests API Postman'
                ])
            }
        }

        stage('Publication sur GitHub') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: "${GIT_CREDENTIALS}",
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    powershell '''
                        $env:GIT_SSH_COMMAND = "ssh -i `"$env:SSH_KEY`" -o StrictHostKeyChecking=no"

                        git config user.name "Jenkins"
                        git config user.email "jenkins@localhost"

                        git add reports/newman/report.html

                        git diff --cached --quiet

                        if ($LASTEXITCODE -ne 0) {
                            git commit -m "Rapport Newman - Build $env:BUILD_NUMBER"
                            git push origin HEAD:main
                        }
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminé.'
        }
    }
}
