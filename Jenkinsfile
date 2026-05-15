pipeline {
  
    agent {
        docker { image 'rockylinux/rockylinux:10.1.2025112' }
    }

    options {
        timeout(time: 4, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '5'))
        timestamps()
        disableConcurrentBuilds()
    }
  
    parameters {
        booleanParam(
            name: 'RUN_SLOW_TESTS',
            defaultValue: false,
            description: 'Include @Slow-annotated tests (significantly increases build time)'
        )
        booleanParam(
            name: 'RUN_BEAST_TESTS',
            defaultValue: true,
            description: 'Run tests in beast mode (repeat each test multiple times)'
        )
        string(
            name: 'BEAST_COUNT',
            defaultValue: '5',
            description: 'Number of repetitions per test in beast mode'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Build only, skip all tests (useful for artifact-only builds)'
        )
    }

    environment {
        SOLR_VERSION = '10.0.0'
        ZOOKEEPER_VERSION = '3.9.5'
        TEMURIN_MAJOR_VERSION = '21'
        TEMURIN_VERSION = "${env.TEMURIN_MAJOR_VERSION}.0.11"
        TEMURIN_PATCH_RELEASE = '10'
        JAVA_HOME = "/var/lib/jenkins/workspace/solr-cicd/java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}"
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
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
                expression { return !fileExists("downloads/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz") }
            }
            steps {
                    echo "HTTP Getting Temurin JDK"
                    httpRequest(
                        url: "https://github.com/adoptium/temurin${env.TEMURIN_MAJOR_VERSION}-binaries/releases/download/jdk-${env.TEMURIN_VERSION}%2B${env.TEMURIN_PATCH_RELEASE}/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz",
                        outputFile: "downloads/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz"
                    )

            }
        }

        stage('Extract Temurin JDK') {
           when {
                expression { return !fileExists("java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}") }
            }
            steps {
                // Extracts the file into the specified directory
                untar file: "downloads/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz", dir: 'java'
            }
        }

        
        stage('Get ZooKeeper') {
            when {
                expression { return !fileExists("downloads/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz") }
            }
            steps {
                    echo "HTTP Getting ZooKeeper"
                    httpRequest(
                        url: "https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-${env.ZOOKEEPER_VERSION}/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz",
                        outputFile: "downloads/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz"
                    )

            }
        }
      
        stage('Extract ZooKeeper') {
           when {
                expression { return !fileExists("src/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz") }
            }
            steps {
                // Extracts the file into the specified directory
                untar file: "downloads/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz", dir: 'src'
            }
        }

      
        stage('Get Solr') {
            when {
                expression { return !fileExists("downloads/solr-${env.SOLR_VERSION}.tgz") }
            }
            steps {
                    echo "HTTP Getting Solr Source Code"
                    httpRequest(
                        url: "https://www.apache.org/dyn/closer.lua/solr/solr/${env.SOLR_VERSION}/solr-${env.SOLR_VERSION}.tgz?action=download",
                        outputFile: "downloads/solr-${env.SOLR_VERSION}.tgz"
                    )

            }
        }

        stage('Extract Solr') {
           when {
                expression { return !fileExists("src/solr-${env.SOLR_VERSION}") }
            }
            steps {
                // Extracts the file into the specified directory
                untar file: "downloads/solr-${env.SOLR_VERSION}.tgz", dir: 'src'
            }
        }

        stage('Validate Environment') {
            steps {
                sh '$JAVA_HOME/bin/java -version'
            }
        }


}
