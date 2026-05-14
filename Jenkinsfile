pipeline {
  
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        JAVA_VERSION = '21.0.2'
        JAVA_HOME = "/var/lib/jenkins/workspace/solr-cicd/java/jdk-${env.JAVA_VERSION}/"
        // Update PATH to include the new JAVA_HOME/bin
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
        SOLR_VERSION = '10.0.0'
    }

    stages {

        stage('Create Directories') {
            steps {
                // Creates any missing parent directories
                sh 'mkdir -p downloads'
                sh 'mkdir -p java'
                sh 'mkdir -p src'
            }
        }
      
        stage('Download Linux OpenJDK') {
            when {
                expression { return !fileExists("downloads/openjdk-${env.JAVA_VERSION}_linux-x64_bin.tar.gz") }
            }
            steps {
                    echo "HTTP Getting Linux OpenJDK"
                    httpRequest(
                        url: "https://download.java.net/java/GA/jdk${env.JAVA_VERSION}/f2283984656d49d69e91c558476027ac/13/GPL/openjdk-${env.JAVA_VERSION}_linux-x64_bin.tar.gz",
                        outputFile: "downloads/openjdk-${env.JAVA_VERSION}_linux-x64_bin.tar.gz"
                    )

            }
        }

        stage('Extract Linux OpenJDK') {
            steps {
                // Extracts the file into the specified directory
                untar file: "downloads/openjdk-${env.JAVA_VERSION}_linux-x64_bin.tar.gz", dir: 'java'
            }
        }

      
        stage('Get Solr Source Code') {
            when {
                expression { return !fileExists("downloads/solr-${env.SOLR_VERSION}-src.tgz") }
            }
            steps {
                    echo "HTTP Getting Solr Source Code"
                    httpRequest(
                        url: "https://www.apache.org/dyn/closer.lua/solr/solr/${env.SOLR_VERSION}/solr-${env.SOLR_VERSION}-src.tgz?action=download",
                        outputFile: "downloads/solr-${env.SOLR_VERSION}-src.tgz"
                    )

            }
        }

     

        stage('Extract Solr Source Code') {
            steps {
                // Extracts the file into the specified directory
                untar file: "downloads/solr-${env.SOLR_VERSION}-src.tgz", dir: 'src'
            }
        }

        stage('Build a full binary distribution') {
            steps {
                dir("src/solr-${env.SOLR_VERSION}") {
                    sh 'pwd'
                    sh './gradlew assemble'
                }
                
            }
        }

        stage('Test') {
            steps {
                dir("src/solr-${env.SOLR_VERSION}") {
                    // Runs all tests
                    sh './gradlew test'

                    // Or run specific module tests, e.g., for Solr Core
                    // sh './gradlew :solr:core:test'
                }
            }
        }



      
    }

    post {
        always {
            dir("src/solr-${env.SOLR_VERSION}") {
                junit '**/build/test-results/**/*.xml'
            }
        }
    }
}
