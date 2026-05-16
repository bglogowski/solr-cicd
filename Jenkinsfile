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

                sh 'mkdir -p java'
                sh 'mkdir -p src'
            }
        }
            
        stage('Import Temurin Java GPG Keys') {
            when {
                expression { return !fileExists("java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}") }
            }
            steps {
                sh 'wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | gpg --import'
            }
        }
        
        stage('Download Temurin Java') {
            when {
                expression { return !fileExists("downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz") }
            }
            steps {
                sh 'mkdir -p downloads/temurin'
                dir('downloads/temurin') {
                    echo "Getting Temurin JDK binary distribution"
                    sh """
                        wget -q \
                            -O OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz
                            https://github.com/adoptium/temurin${env.TEMURIN_MAJOR_VERSION}-binaries/releases/download/jdk-${env.TEMURIN_VERSION}%2B${env.TEMURIN_PATCH_RELEASE}/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz
                    """
                      
                    echo "Getting Temurin JDK binary distribution signature"
                    sh """
                        wget -q \
                            -O OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz.sig
                            https://github.com/adoptium/temurin${env.TEMURIN_MAJOR_VERSION}-binaries/releases/download/jdk-${env.TEMURIN_VERSION}%2B${env.TEMURIN_PATCH_RELEASE}/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz.sig
                    """

                }
            }
        }

        stage('Validate Temurin Java files') {
           when {
                expression { return !fileExists("java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}") }
            }
            steps {
                echo "Verify GPG signature"
                dir('downloads/temurin') {
                    sh """
                        gpg --verify \
                            OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz.sig \
                            OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz \
                            || /bin/rm -f OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz
                    """
                }
            }
        }

        stage('Extract Temurin Java') {
           when {
                expression { return !fileExists("java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}") }
            }
            steps {
                sh 'mkdir -p java'
                dir('java') {
                    sh "tar xzf ../downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz"
                }
            }
        }

        stage('Import ZooKeeper GPG Keys') {
            when {
                expression { return !fileExists("src/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin") }
            }
            steps {
                sh 'wget -qO - https://downloads.apache.org/zookeeper/KEYS | gpg --import'
            }
        }

        stage('Download ZooKeeper') {
            when {
                expression { return !fileExists("downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz") }
            }
            steps {
                sh 'mkdir -p downloads/zookeeper'
                dir('downloads/zookeeper') {
                    echo "Getting ZooKeeper binary distribution"
                    sh """
                        wget -q \
                            -O apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz
                            https://dlcdn.apache.org/zookeeper/stable/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz
                    """
              
                    echo "Getting ZooKeeper binary distribution signature"
                    sh """
                        wget -q \
                            -O https://dlcdn.apache.org/zookeeper/stable/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz.asc
                            https://dlcdn.apache.org/zookeeper/stable/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz.asc
                    """
                }
              
            }
        }

        stage('Validate ZooKeeper files') {
           when {
                expression { return !fileExists("src/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin") }
            }
            steps {
                echo "Verify GPG signature"
                dir('downloads/zookeeper') {
                    sh """
                        gpg --verify \
                            apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz.asc \
                            apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz \
                            || /bin/rm -f apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz
                    """
                }
            }
        }
      
        stage('Extract ZooKeeper') {
           when {
                expression { return !fileExists("src/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin") }
            }
            steps {
                sh 'mkdir -p src'
                dir('src') {
                    sh "tar xzf ../downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz"
                }
            }
        }

        stage('Import Solr GPG Keys') {
            when {
                expression { return !fileExists("src/solr-${env.SOLR_VERSION}") }
            }
            steps {
                sh 'wget -qO - https://downloads.apache.org/solr/KEYS | gpg --import'
            }
        }

        stage('Download Solr') {
            when {
                expression { return !fileExists("downloads/solr/solr-${env.SOLR_VERSION}.tgz") }
            }
            steps {
                sh 'mkdir -p downloads/solr'
                dir('downloads/solr') {
                    echo "Getting Solr binary distribution"
                    sh """
                        wget -q \
                            -O solr-${env.SOLR_VERSION}.tgz
                            https://downloads.apache.org/solr/solr/${env.SOLR_VERSION}/solr-${env.SOLR_VERSION}.tgz
                    """
              
                    echo "Getting Solr binary distribution signature"
                    sh """
                        wget -q \
                            -O solr-${env.SOLR_VERSION}.tgz.asc
                            https://downloads.apache.org/solr/solr/${env.SOLR_VERSION}/solr-${env.SOLR_VERSION}.tgz.asc
                    """
                  
                }

            }
        }

        stage('Validate Solr files') {
           when {
                expression { return !fileExists("src/solr-${env.SOLR_VERSION}") }
            }
            steps {
                echo "Verify GPG signature"
                dir('downloads/solr') {
                    sh """
                        gpg --verify \
                            solr-${env.SOLR_VERSION}.tgz.asc \
                            solr-${env.SOLR_VERSION}.tgz \
                            || /bin/rm -f solr-${env.SOLR_VERSION}.tgz
                    """
                }
            }
        }

        stage('Extract Solr') {
           when {
                expression { return !fileExists("src/solr-${env.SOLR_VERSION}") }
            }
            steps {
                sh 'mkdir -p src'
                dir('src') {
                    sh "tar xzf ../downloads/solr/solr-${env.SOLR_VERSION}.tgz"
                }
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
            echo 'Always cleanup temp files.'
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
