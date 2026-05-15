pipeline {
  
    agent any

    options {
        timeout(time: 4, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '5'))
        timestamps()
        disableConcurrentBuilds()
    }
  
    environment {
        SOLR_VERSION = '10.0.0'
        ZOOKEEPER_VERSION = '3.8.6'
        TEMURIN_MAJOR_VERSION = '21'
        TEMURIN_VERSION = "${env.TEMURIN_MAJOR_VERSION}.0.11"
        TEMURIN_PATCH_RELEASE = '10'
        JAVA_HOME = "/var/lib/jenkins/workspace/solr-cicd/java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}"
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Create Directories') {
            steps {
                sh 'mkdir -p downloads/temurin'
                sh 'mkdir -p downloads/zookeeper'
                sh 'mkdir -p downloads/solr'
                sh 'mkdir -p java'
                sh 'mkdir -p src'
            }
        }
            
        stage('Get Temurin Java GPG Keys') {
            when {
                expression { return !fileExists("downloads/temurin/KEYS") }
            }
            steps {
                httpRequest(
                        url: "https://packages.adoptium.net/artifactory/api/gpg/key/public",
                        outputFile: "downloads/temurin/KEYS"
                )
                sh 'gpg --import downloads/temurin/KEYS'
            }
        }
        
        stage('Download Temurin JDK') {
            when {
                expression { return !fileExists("downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz") }
            }
            steps {
                    echo "HTTP Getting Temurin JDK"
                    httpRequest(
                        url: "https://github.com/adoptium/temurin${env.TEMURIN_MAJOR_VERSION}-binaries/releases/download/jdk-${env.TEMURIN_VERSION}%2B${env.TEMURIN_PATCH_RELEASE}/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz",
                        outputFile: "downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz"
                    )
                    httpRequest(
                        url: "https://github.com/adoptium/temurin${env.TEMURIN_MAJOR_VERSION}-binaries/releases/download/jdk-${env.TEMURIN_VERSION}%2B${env.TEMURIN_PATCH_RELEASE}/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz.sig",
                        outputFile: "downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz.sig"
                    )

            }
        }

        stage('Extract Temurin JDK') {
           when {
                expression { return !fileExists("java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}") }
            }
            steps {
                sh """
                    gpg --verify \
                        downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz.sig \
                        downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz \
                        || rm -f downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz
                """
                untar file: "downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz", dir: 'java'
            }
        }


        stage('Get Apache ZooKeeper GPG Keys') {
            when {
                expression { return !fileExists("downloads/zookeeper/KEYS") }
            }
            steps {
                httpRequest(
                        url: "https://downloads.apache.org/zookeeper/KEYS",
                        outputFile: "downloads/zookeeper/KEYS"
                )
                sh 'gpg --import KEYS'
            }
        }


        stage('Get ZooKeeper') {
            when {
                expression { return !fileExists("downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz") }
            }
            steps {
                    echo "HTTP Getting ZooKeeper"
                    httpRequest(
                        url: "https://dlcdn.apache.org/zookeeper/stable/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz",
                        outputFile: "downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz"
                    )
              
                    httpRequest(
                        url: "https://dlcdn.apache.org/zookeeper/stable/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz.asc",
                        outputFile: "downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz.asc"
                    )
              
            }
        }
      
        stage('Extract ZooKeeper') {
           when {
                expression { return !fileExists("src/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin") }
            }
            steps {
                sh """
                    gpg --verify \
                        downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz.asc \
                        downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz \
                        || rm -f downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz
                """
                untar file: "downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz", dir: 'src'
            }
        }

        stage('Get Apache Solr GPG Keys') {
            when {
                expression { return !fileExists("downloads/solr/KEYS") }
            }
            steps {
                httpRequest(
                        url: "https://downloads.apache.org/solr/KEYS",
                        outputFile: "downloads/solr/KEYS"
                )
                sh 'gpg --import downloads/solr/KEYS'
            }
        }

        stage('Get Solr') {
            when {
                expression { return !fileExists("downloads/solr/solr-${env.SOLR_VERSION}.tgz") }
            }
            steps {
                    echo "HTTP Getting Solr Source Code"
                    httpRequest(
                        url: "https://www.apache.org/dyn/closer.lua/solr/solr/${env.SOLR_VERSION}/solr-${env.SOLR_VERSION}.tgz?action=download",
                        outputFile: "downloads/solr/solr-${env.SOLR_VERSION}.tgz"
                    )
              
                    httpRequest(
                        url: "https://downloads.apache.org/solr/solr/${env.SOLR_VERSION}/solr-${env.SOLR_VERSION}.tgz.asc",
                        outputFile: "downloads/solr/solr-${env.SOLR_VERSION}.tgz.asc"
                    )

            }
        }

        stage('Extract Solr') {
           when {
                expression { return !fileExists("src/solr-${env.SOLR_VERSION}") }
            }
            steps {
                sh """
                    gpg --verify \
                        downloads/solr/solr-${env.SOLR_VERSION}.tgz.asc \
                        downloads/solr/solr-${env.SOLR_VERSION}.tgz \
                        || rm -f downloads/solr/solr-${env.SOLR_VERSION}.tgz
                """
                untar file: "downloads/solr-${env.SOLR_VERSION}.tgz", dir: 'src'
            }
        }

        stage('Validate Environment') {
            steps {
                sh '$JAVA_HOME/bin/java -version'
                dir("src/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin") {
                    sh '$JAVA_HOME/bin/java -cp "lib/*" org.apache.zookeeper.Version'
                }
                sh "src/solr-${env.SOLR_VERSION}/bin/solr version"
                
            }
        }

    }
    post {
        always {
            // cleanWs()
        }
        success {
            echo 'Build was successful!'
        }
        failure {
            echo 'Build failed.'
        }
    }

}
