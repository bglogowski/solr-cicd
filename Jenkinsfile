pipeline {
  
    agent any

    options {
        disableConcurrentBuilds()
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
                expression { return !fileExists('downloads/openjdk-26.0.1_linux-x64_bin.tar.gz') }
            }
            steps {
                    echo "HTTP Getting Linux OpenJDK"
                    httpRequest(
                        url: 'https://download.java.net/java/GA/jdk26.0.1/458fda22e4c54d5ba572ab8d2b22eb83/8/GPL/openjdk-26.0.1_linux-x64_bin.tar.gz',
                        outputFile: 'downloads/openjdk-26.0.1_linux-x64_bin.tar.gz'
                    )

            }
        }

        stage('Extract Linux OpenJDK') {
            steps {
                // Extracts the file into the specified directory
                untar file: 'downloads/openjdk-26.0.1_linux-x64_bin.tar.gz', dir: 'java'
            }
        }

      
        stage('Get Solr Source Code') {
            when {
                expression { return !fileExists('downloads/solr-10.0.0-src.tgz') }
            }
            steps {
                    echo "HTTP Getting Solr Source Code"
                    httpRequest(
                        url: 'https://www.apache.org/dyn/closer.lua/solr/solr/10.0.0/solr-10.0.0-src.tgz?action=download',
                        outputFile: 'downloads/solr-10.0.0-src.tgz'
                    )

            }
        }

     

        stage('Extract Solr Source Code') {
            steps {
                // Extracts the file into the specified directory
                untar file: 'downloads/solr-10.0.0-src.tgz', dir: 'src'
            }
        }
    }
}
