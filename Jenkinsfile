pipeline {
    agent any

    stages {
        stage('Run Newman Tests with Reports') {
            steps {
                bat 'newman run postman-collection.json -e postman-environment.json --reporters cli,htmlextra --reporter-htmlextra-export report.html'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'report.html', onlyIfSuccessful: true
                }
            }
        }
    }
}