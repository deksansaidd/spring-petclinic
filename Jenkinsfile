pipeline {
    agent any

    triggers {
        // Trigger every 5 minutes, only on Mondays (day 2 in cron = Monday)
        cron('H/5 * * * 2')
    }

    tools {
        // Ensure you have a JDK named 'JDK17' configured in Jenkins Global Tool Config
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        // Artifact output directory
        ARTIFACT_DIR = 'target'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '===== Checking out source code ====='
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo '===== Building the application ====='
                sh 'mvn clean compile -DskipTests'
            }
        }

        stage('Test & Code Coverage (JaCoCo)') {
            steps {
                echo '===== Running tests and generating JaCoCo coverage report ====='
                sh 'mvn test jacoco:report'
            }
            post {
                always {
                    // Publish JaCoCo HTML report in Jenkins
                    jacoco(
                        execPattern: '**/target/jacoco.exec',
                        classPattern: '**/target/classes',
                        sourcePattern: '**/src/main/java',
                        exclusionPattern: '**/test/**',
                        minimumInstructionCoverage: '0',
                        minimumBranchCoverage:      '0',
                        minimumComplexityCoverage:  '0',
                        minimumLineCoverage:        '0',
                        minimumMethodCoverage:      '0',
                        minimumClassCoverage:       '0'
                    )
                }
            }
        }

        stage('Package (Generate Artifact)') {
            steps {
                echo '===== Packaging application into JAR artifact ====='
                sh 'mvn package -DskipTests'
            }
            post {
                success {
                    // Archive the JAR so it's downloadable from Jenkins
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Verify Artifact') {
            steps {
                echo '===== Listing generated artifacts ====='
                sh 'ls -lh target/*.jar'
            }
        }
    }

    post {
        always {
            echo '===== Pipeline finished ====='
        }
        success {
            echo '===== BUILD SUCCESSFUL — Artifact and JaCoCo report are available ====='
        }
        failure {
            echo '===== BUILD FAILED — Check logs above ====='
        }
    }
}
