pipeline {
  
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        TEMURIN_VERSION = '21.0.11'
        TEMURIN_PATCH_RELEASE = '10'
        JAVA_HOME = "/var/lib/jenkins/workspace/solr-cicd/java/jdk-${env.JAVA_VERSION}/"
        // Update PATH to include the new JAVA_HOME/bin
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
        SOLR_VERSION = '10.0.0'
        // SOLR_OPTS = "${env.SOLR_OPTS} -Djava.locale.providers=COMPAT,CLDR,SPI"
        JAVA_TOOL_OPTIONS = '-Djava.locale.providers=COMPAT,CLDR,SPI --add-opens=java.base/java.lang=ALL-UNNAMED'
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
      
        stage('Download Temurin JDK') {
            when {
                expression { return !fileExists("downloads/OpenJDK21U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz") }
            }
            steps {
              // https://github.com/adoptium/temurin21-binaries/releases/download/jdk-21.0.11%2B10/OpenJDK21U-jdk_x64_linux_hotspot_21.0.11_10.tar.gz
                    echo "HTTP Getting Temurin JDK"
                    httpRequest(
                        url: "https://github.com/adoptium/temurin21-binaries/releases/download/jdk-${env.TEMURIN_VERSION}%2B${env.TEMURIN_PATCH_RELEASE}/OpenJDK21U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz",
                        outputFile: "downloads/OpenJDK21U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz"
                    )

            }
        }

        stage('Extract Linux OpenJDK') {
           when {
                expression { return !fileExists("java/jdk-${env.TEMURIN_VERSION}") }
            }
            steps {
                // Extracts the file into the specified directory
                untar file: "downloads/OpenJDK21U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz", dir: 'java'
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
           when {
                expression { return !fileExists("src/solr-${env.SOLR_VERSION}") }
            }
            steps {
                // Extracts the file into the specified directory
                untar file: "downloads/solr-${env.SOLR_VERSION}-src.tgz", dir: 'src'
            }
        }

        stage('Build a full binary distribution') {
            when {
                expression { return !fileExists("src/solr-${env.SOLR_VERSION}/solr/packaging/build/distributions/solr-${env.SOLR_VERSION}-SNAPSHOT.tgz") }
            }
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

      // src/solr-10.0.0/solr/packaging/build/distributions/solr-10.0.0-SNAPSHOT.tgz
      // src/solr-10.0.0/solr/packaging/build/distributions/solr-10.0.0-SNAPSHOT-slim.tgz


      
    }

    post {
        always {
            dir("src/solr-${env.SOLR_VERSION}") {
                junit '**/build/test-results/**/*.xml'
                archiveArtifacts artifacts: 'solr/packaging/build/distributions/*.tgz', onlyIfSuccessful: true
            }
        }
    }
}
