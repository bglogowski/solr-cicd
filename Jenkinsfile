pipeline {
  
    agent any

    options {
        timeout(time: 4, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '5'))
        timestamps()
        disableConcurrentBuilds()
    }
  
    parameters {
        string(
            name: 'SOLR_BRANCH',
            defaultValue: 'main',
            description: 'Branch or tag to build (e.g. main, branch_9x, releases/solr/9.7.0)'
        )
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
        JAVA_TOOL_OPTIONS = '-Xmx2g -XX:+HeapDumpOnOutOfMemoryError'
        TEMURIN_MAJOR_VERSION = '21'
        TEMURIN_VERSION = "${env.TEMURIN_MAJOR_VERSION}.0.11"
        TEMURIN_PATCH_RELEASE = '10'
        JAVA_HOME = "/var/lib/jenkins/workspace/solr-cicd/java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}/"
        // Update PATH to include the new JAVA_HOME/bin
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
        SOLR_VERSION = '10.0.0'
        // SOLR_OPTS = "${env.SOLR_OPTS} -Djava.locale.providers=COMPAT,CLDR,SPI"
        // JAVA_TOOL_OPTIONS = '-Djava.locale.providers=COMPAT,CLDR,SPI --add-opens=java.base/java.lang=ALL-UNNAMED'
        GRADLE_OPTS     = '-Dorg.gradle.daemon=false -Dorg.gradle.workers.max=4'
        GRADLE_USER_HOME = "${WORKSPACE}/.gradle-home"
             // Solr test settings
        TEST_OPTS = [
            '-Dtests.multiplier=1',
            '-Dtests.nightly=false',
            '-Dtests.weekly=false',
            '-Dtests.badapples=false',
            "-Dtests.slow=${params.RUN_SLOW_TESTS}",
            '-Dtests.asserts=true',
            '-Dtests.timeoutSuite=900000',   // 15 min per suite
        ].join(' ')

        // Randomized testing seed (fixed per build for reproducibility)
        TESTS_SEED = "${currentBuild.startTimeInMillis}"
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
              // https://github.com/adoptium/temurin21-binaries/releases/download/jdk-21.0.11%2B10/OpenJDK21U-jdk_x64_linux_hotspot_21.0.11_10.tar.gz
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

        stage('Validate Environment') {
            steps {
                sh '$JAVA_HOME/bin/java -version'
                dir("src/solr-${env.SOLR_VERSION}") {
                    sh './gradlew --version'
                    sh './gradlew javaToolchains || true'
                }
            }
        }

        stage('Build a full binary distribution') {
            when {
                expression { return !fileExists("src/solr-${env.SOLR_VERSION}/solr/packaging/build/distributions/solr-${env.SOLR_VERSION}-SNAPSHOT.tgz") }
            }
            steps {
                dir("src/solr-${env.SOLR_VERSION}") {
                    sh 'pwd'
                    sh './gradlew clean'
                    sh './gradlew assemble'
                }
                
            }
            post {
                failure {
                    echo 'Compilation failed — archiving any error logs.'
                    archiveArtifacts artifacts: '**/build/tmp/**/*.log', allowEmptyArchive: true
                }
            }
        }

        stage('Run Tests') {
            parallel {
            steps {
              stage('Core Tests') {
                dir("src/solr-${env.SOLR_VERSION}") {
                    // Runs all tests
                    sh """
                        ./gradlew \
                            :solr:core:test \
                                ${env.TEST_OPTS} \
                                -Dtests.seed=${env.TESTS_SEED} \
                                ${params.RUN_BEAST_TESTS ? "-Dtests.iters=${params.BEAST_COUNT}" : ''} \
                                --continue \
                                -PtestJvmArgs='-Xmx1g'
                        """

                    // Or run specific module tests, e.g., for Solr Core
                    // sh './gradlew :solr:core:test'
                }
              }

                   stage('Solrj Tests') {
                    steps {
                        sh """
                            ./gradlew \
                                :solr:solrj:test \
                                :solr:solrj-streaming:test \
                                :solr:solrj-zookeeper:test \
                                ${env.TEST_OPTS} \
                                -Dtests.seed=${env.TESTS_SEED} \
                                --continue \
                                -PtestJvmArgs='-Xmx1g'
                        """
                    }
                }

                stage('Module Tests') {
                    steps {
                        sh """
                            ./gradlew \
                                :solr:modules:analysis-extras:test \
                                :solr:modules:clustering:test \
                                :solr:modules:cross-dc:test \
                                :solr:modules:extraction:test \
                                :solr:modules:gcs-repository:test \
                                :solr:modules:hdfs:test \
                                :solr:modules:jwt-auth:test \
                                :solr:modules:langid:test \
                                :solr:modules:ltr:test \
                                :solr:modules:opentelemetry:test \
                                :solr:modules:s3-repository:test \
                                :solr:modules:scripting:test \
                                :solr:modules:sql:test \
                                ${env.TEST_OPTS} \
                                -Dtests.seed=${env.TESTS_SEED} \
                                --continue \
                                -PtestJvmArgs='-Xmx1g'
                        """
                    }
                }

                stage('Test Framework / API Tests') {
                    steps {
                        sh """
                            ./gradlew \
                                :solr:test-framework:test \
                                :solr:api:test \
                                ${env.TEST_OPTS} \
                                -Dtests.seed=${env.TESTS_SEED} \
                                --continue \
                                -PtestJvmArgs='-Xmx512m'
                        """
                    }
                }


              
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
