// pipeline {
//     //  This section defines the execution environment for your build, test, and deployment tasks.
//     agent {
//         docker {
//             image 'maven:3.8.5-openjdk-17'
//             args '-v $HOME/.m2:/root/.m2'
//         }

//     // The options directive allows configuring Pipeline-specific options from within the Pipeline itself.
//     options {
//         // Set a timeout period for the Pipeline run, after which Jenkins should abort the Pipeline.
//         timeout(time: 30, unit: 'MINUTES')

//         // Persist artifacts and console output for the specific number of recent Pipeline runs.
//         buildDiscarder(logRotator(numToKeepStr: '10'))

//         // Disallow concurrent executions of the Pipeline.
//         // Can be useful for preventing simultaneous accesses to shared resources
//         disableConcurrentBuilds()
//     }

//     environment {
//         // Application Configuration
//         APP_NAME = 'microservice2'
//         VERSION = readMavenPom().getVersion()

//         // Git Configuration
//         GIT_CREDENTIALS_ID = 'learn-jenkins-credential-id'
//         GITOPS_BRANCH = 'gitops'
//         GITOPS_REPO = 'https://github.com/learning-jenkins/microservice-2.git'

//         // // Registry Configuration
//         // DOCKER_REGISTRY = 'your-registry.com'
//         // DOCKER_CREDENTIALS_ID = 'your-docker-credentials'
//     }

//     // Containing a sequence of one or more stage directives, the stages section is
//     // where the bulk of the "work" described by a Pipeline will be located.
//     stages {
//         stage('Setup') {
//             steps {
//                 powershell '''
//                     podman machine start
//                     Start-Sleep -Seconds 3
//                     Start-Process -NoNewWindow -FilePath "podman" -ArgumentList "system service -t 0"
//                     [System.Environment]::SetEnvironmentVariable('DOCKER_HOST', 'npipe:////./pipe/podman-pipe', 'Machine')
//                     $env:DOCKER_HOST = 'npipe:////./pipe/podman-pipe'
//                 '''
//             }
//         }

//         stage('Checkout') {
//             steps {
//                 checkout([
//                     $class: 'GitSCM',
//                     branches: [[name: 'gitops']], // Changed from main to gitops
//                     extensions: [
//                         [
//                             $class: 'CleanCheckout'
//                         ],
//                         [
//                             $class: 'RelativeTargetDirectory',
//                             relativeTargetDir: 'app-source'
//                         ]
//                     ],
//                     userRemoteConfigs: [[
//                         credentialsId: env.GIT_CREDENTIALS_ID,
//                         url: 'https://github.com/learning-jenkins/microservice-2.git'
//                     ]]
//                 ])

//                 dir('app-source') {
//                     script {
//                         // Get current commit hash for tagging
//                         COMMIT_HASH = sh(
//                             script: 'git rev-parse --short HEAD',
//                             returnStdout: true
//                         ).trim()

//                         // Set display name with version and commit
//                         currentBuild.displayName = "${env.VERSION}-${COMMIT_HASH}"

//                         // Optional: Verify available branches
//                         echo "Available branches in repository:"
//                         sh 'git branch -a'
//                     }
//                 }
//             }
//         }

//         stage('Linting') {
//             steps {
//                 dir('app-source') {
//                     script {
//                         // Checkstyle linting
//                         sh 'mvn checkstyle:checkstyle'

//                         // PMD static analysis
//                         sh 'mvn pmd:pmd'

//                         // SpotBugs analysis
//                         sh 'mvn spotbugs:spotbugs'
//                     }
//                 }
//             }
//             post {
//                 always {
//                     dir('app-source') {
//                         // Archive code quality reports
//                         recordIssues(
//                             tools: [
//                                 checkStyle(pattern: '**/checkstyle-result.xml'),
//                                 pmdParser(pattern: '**/pmd.xml'),
//                                 spotBugs(pattern: '**/spotbugsXml.xml')
//                             ]
//                         )
//                     }
//                 }
//             }
//         }

//         stage('Build') {
//             steps {
//                 dir('app-source') {
//                     script {
//                         // Clean build with tests skipped (they run in test stage)
//                         sh 'mvn clean compile -DskipTests'

//                         // Package the application
//                         sh 'mvn package -DskipTests'

//                         // Store the built artifact
//                         archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
//                     }
//                 }
//             }
//         }

//         stage('Test') {
//             steps {
//                 dir('app-source') {
//                     script {
//                         // Run unit tests
//                         sh 'mvn test'

//                         // Run integration tests if any
//                         sh 'mvn verify -DskipUnitTests'
//                     }
//                 }
//             }
//             post {
//                 always {
//                     dir('app-source') {
//                         // Publish test results
//                         junit '**/target/surefire-reports/*.xml'

//                         // Publish integration test results
//                         junit '**/target/failsafe-reports/*.xml'

//                         // Generate and publish JaCoCo coverage report
//                         jacoco(
//                             execPattern: '**/target/jacoco.exec',
//                             classPattern: '**/target/classes',
//                             sourcePattern: '**/src/main/java'
//                         )
//                     }
//                 }
//             }
//         }

//         // stage('Update GitOps Tag') {
//         //     steps {
//         //         script {
//         //             // Checkout GitOps repository
//         //             dir('gitops-repo') {
//         //                 checkout([
//         //                     $class: 'GitSCM',
//         //                     branches: [[name: "*/${env.GITOPS_BRANCH}"]],
//         //                     extensions: [
//         //                         [
//         //                             $class: 'CleanCheckout'
//         //                         ],
//         //                         [
//         //                             $class: 'UserIdentity',
//         //                             email: 'jenkins@your-company.com',
//         //                             name: 'Jenkins CI'
//         //                         ]
//         //                     ],
//         //                     userRemoteConfigs: [[
//         //                         credentialsId: env.GIT_CREDENTIALS_ID,
//         //                         url: env.GITOPS_REPO
//         //                     ]]
//         //                 ])

//         //                 // Update the image tag in deployment manifest
//         //                 def manifestFile = 'apps/your-springboot-app/deployment.yaml'
//         //                 if (fileExists(manifestFile)) {
//         //                     def manifest = readYaml file: manifestFile

//         //                     // Update image tag - adjust path based on your manifest structure
//         //                     manifest.spec.template.spec.containers[0].image =
//         //                         "${env.DOCKER_REGISTRY}/${env.APP_NAME}:${env.VERSION}-${COMMIT_HASH}"

//         //                     // Write updated manifest
//         //                     writeYaml file: manifestFile, data: manifest

//         //                     // Commit and push changes
//         //                     sh """
//         //                         git add ${manifestFile}
//         //                         git commit -m "Update ${env.APP_NAME} to version ${env.VERSION}-${COMMIT_HASH}"
//         //                         git push origin ${env.GITOPS_BRANCH}
//         //                     """
//         //                 } else {
//         //                     error "GitOps manifest file not found: ${manifestFile}"
//         //                 }
//         //             }
//         //         }
//         //     }
//         // }
//     }

//     // The post section defines one or more additional steps that are run upon the completion
//     // of a Pipeline’s or stage’s run (depending on the location of the post section within the Pipeline).
//     post {
//         always {
//             // Clean workspace
//             cleanWs()

//             // Send notifications based on build status
//             script {
//                 def buildStatus = currentBuild.result ?: 'SUCCESS'
//                 echo "Build finished with status: ${buildStatus}"
//             }
//         }
//         success {
//             emailext (
//                 subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
//                 body: """
//                 Spring Boot Application Build Successful!

//                 Application: ${env.APP_NAME}
//                 Version: ${env.VERSION}
//                 Commit: ${COMMIT_HASH}
//                 Build URL: ${env.BUILD_URL}
//                 """,
//                 to: 'example@example.com'
//             )
//         }
//         failure {
//             emailext (
//                 subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
//                 body: """
//                 Spring Boot Application Build Failed!

//                 Application: ${env.APP_NAME}
//                 Version: ${env.VERSION}
//                 Commit: ${COMMIT_HASH}
//                 Build URL: ${env.BUILD_URL}

//                 Please check the build logs for details.
//                 """,
//                 to: 'example@example.com'
//             )
//         }
//     }
// }


pipeline {
    agent any

    stages {
        stage('Lint') {
            steps {
                // sh './mvnw spotless:check' This works for Jenkins on Linux agent.
                bat 'mvnw.cmd spotless:check'
            }
        }

        stage('Build') {
            steps {
                // sh './mvnw clean package' This works for Jenkins on Linux agent.
                bat 'mvnw.cmd clean package'
            }
        }
    }
}