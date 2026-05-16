pipeline {
  
    agent any

    options {
        ansiColor('xterm')
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
        
        stage('Download Temurin Java') {
            when {
                expression { return !fileExists("downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz") }
            }
            steps {
                sh 'mkdir -p downloads/temurin'
                dir('downloads/temurin') {
                    echo 'Get Temurin Java binary distribution'
                    sh """
                        wget -q \
                            -O OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz \
                            https://github.com/adoptium/temurin${env.TEMURIN_MAJOR_VERSION}-binaries/releases/download/jdk-${env.TEMURIN_VERSION}%2B${env.TEMURIN_PATCH_RELEASE}/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz
                    """
                      
                    echo 'Get Temurin Java binary distribution signature'
                    sh """
                        wget -q \
                            -O OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz.sig \
                            https://github.com/adoptium/temurin${env.TEMURIN_MAJOR_VERSION}-binaries/releases/download/jdk-${env.TEMURIN_VERSION}%2B${env.TEMURIN_PATCH_RELEASE}/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz.sig
                    """

                }
            }
        }


        stage('Validate and Extract Temurin Java') {
           when {
                expression { return !fileExists("java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}") }
            }
            steps {
                echo 'Import Temurin Java GPG public key(s)'
                sh 'wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | gpg --import'
              
                echo 'Verify Temurin Java GPG signature'
                dir('downloads/temurin') {
                    sh """
                        gpg --verify \
                            OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz.sig \
                            OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz \
                            || /bin/rm -f OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz
                    """
                }
              
                echo 'Extract Temurin Java'
                sh 'mkdir -p java'
                dir('java') {
                    sh "tar xzf ../downloads/temurin/OpenJDK${env.TEMURIN_MAJOR_VERSION}U-jdk_x64_linux_hotspot_${env.TEMURIN_VERSION}_${env.TEMURIN_PATCH_RELEASE}.tar.gz"
                }
            }
        }


        stage('Download ZooKeeper') {
            // Note: This should be the latest stable release
          
            when {
                expression { return !fileExists("downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz") }
            }
            steps {
                sh 'mkdir -p downloads/zookeeper'
                dir('downloads/zookeeper') {
                    echo 'Get ZooKeeper binary distribution'
                    sh """
                        wget -q \
                            -O apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz \
                            https://dlcdn.apache.org/zookeeper/stable/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz
                    """
              
                    echo 'Get ZooKeeper binary distribution signature'
                    sh """
                        wget -q \
                            -O apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz.asc \
                            https://dlcdn.apache.org/zookeeper/stable/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz.asc
                    """
                }
              
            }
        }

      
        stage('Validate and Extract ZooKeeper') {
           when {
                expression { return !fileExists("src/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin") }
            }
            steps {
                echo 'Import Zookeeper GPG public key(s)'
                sh 'wget -qO - https://downloads.apache.org/zookeeper/KEYS | gpg --import'
              
                echo 'Verify Zookeeper GPG signature'
                dir('downloads/zookeeper') {
                    sh """
                        gpg --verify \
                            apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz.asc \
                            apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz
                    """
                }
              
                echo 'Extract Zookeeper'
                sh 'mkdir -p src'
                dir('src') {
                    sh "tar xzf ../downloads/zookeeper/apache-zookeeper-${env.ZOOKEEPER_VERSION}-bin.tar.gz"
                }
            }
        }
      

        stage('Download Solr') {
            when {
                expression { return !fileExists("downloads/solr/solr-${env.SOLR_VERSION}.tgz") }
            }
            steps {
                sh 'mkdir -p downloads/solr'
                dir('downloads/solr') {
                    echo 'Get Solr binary distribution'
                    sh """
                        wget -q \
                            -O solr-${env.SOLR_VERSION}.tgz \
                            https://downloads.apache.org/solr/solr/${env.SOLR_VERSION}/solr-${env.SOLR_VERSION}.tgz
                    """
              
                    echo 'Get Solr binary distribution signature'
                    sh """
                        wget -q \
                            -O solr-${env.SOLR_VERSION}.tgz.asc \
                            https://downloads.apache.org/solr/solr/${env.SOLR_VERSION}/solr-${env.SOLR_VERSION}.tgz.asc
                    """
                  
                }

            }
        }

        stage('Validate and Extract Solr') {
           when {
                expression { return !fileExists("src/solr-${env.SOLR_VERSION}") }
            }
            steps {
                echo 'Import Solr GPG public key(s)'
                sh 'wget -qO - https://downloads.apache.org/solr/KEYS | gpg --import'

                echo 'Verify Solr GPG signature'
                dir('downloads/solr') {
                    sh """
                        gpg --verify \
                            solr-${env.SOLR_VERSION}.tgz.asc \
                            solr-${env.SOLR_VERSION}.tgz \
                            || /bin/rm -f solr-${env.SOLR_VERSION}.tgz
                    """
                }
              
                echo 'Extract Solr'
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
