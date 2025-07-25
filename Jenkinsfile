pipeline {
    agent any
    tools {
        maven 'maven'         // Make sure 'maven' is set as a tool in Jenkins
        jdk 'jdk17'           // Make sure 'jdk17' is set as a tool in Jenkins
        // sonarQube 'sonarscanner' // Uncomment only if you have such a tool mapping
    }

    environment {
        SNAP_REPO      = 'vprofile-snapshot'
        NEXUS_USER     = 'admin'
        NEXUS_PASS     = 'dhiren'
        RELEASE_REPO   = 'vprofile-release'
        CENTRAL_REPO   = 'vpro-maven-central'
        NEXUSIP        = '172.31.80.64'
        NEXUSPORT      = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN    = 'nexuslogin'          // Jenkins credential ID for Nexus repo
        SONARSERVER    = 'sonarserver'         // Jenkins SonarQube server name
        SONARSCANNER   = 'sonarscanner'        // Jenkins tool name for SonarScanner
    }

    // Set BUILD_TIMESTAMP at the start for repeatable builds
    options {
        timestamps()
    }

    stages {
        stage('Init') {
            steps {
                script {
                    env.BUILD_TIMESTAMP = new Date().format('yyyyMMddHHmmss')
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Build completed successfully.'
                    archiveArtifacts artifacts: '**/*.war'
                }
                failure {
                    echo 'Build failed.'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
            post {
                success {
                    echo 'Tests passed successfully.'
                }
                failure {
                    echo 'Tests failed.'
                }
            }
        }
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Checkstyle analysis completed successfully here.'
                }
                failure {
                    echo 'Checkstyle analysis failed here.'
                }
            }
        }
        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool("${SONARSCANNER}")
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=vprofile \
                          -Dsonar.projectVersion=1.0 \
                          -Dsonar.sources=src/ \
                          -Dsonar.java.binaries=target/classes \
                          -Dsonar.junit.reportPaths=target/surefire-reports/ \
                          -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                          -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    """
                }
            }
        }
        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Upload artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [                        
                        artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war'
                    ]
                )
            }
        }
    }
}
