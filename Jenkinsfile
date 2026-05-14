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
                sh 'mkdir -p src'
            }
        }
    
        stage('Get Solr Source Code') {
            when {
                expression { return !fileExists('downloads/solr-10.0.0-src.tgz') }
            }
            steps {
                    echo "Getting Solr Source Code"
                    httpRequest(
                        url: 'https://www.apache.org/dyn/closer.lua/solr/solr/10.0.0/solr-10.0.0-src.tgz?action=download',
                        outputFile: 'downloads/solr-10.0.0-src.tgz'
                    )

            }
        }

     

        stage('Extract') {
            steps {
                // Extracts the file into the specified directory
                untar file: 'downloads/solr-10.0.0-src.tgz', dir: 'src'
            }
        }
    }
}
