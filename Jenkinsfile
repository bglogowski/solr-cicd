pipeline {
  
    agent any

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
      
        TEMURIN_MAJOR_VERSION = '21'
        TEMURIN_VERSION = "${env.TEMURIN_MAJOR_VERSION}.0.11"
        TEMURIN_PATCH_RELEASE = '10'
        JAVA_HOME = "/var/lib/jenkins/workspace/solr-cicd/java/jdk-${env.TEMURIN_VERSION}+${env.TEMURIN_PATCH_RELEASE}"
        JAVA_TOOL_OPTIONS = '-Xmx2g -XX:+HeapDumpOnOutOfMemoryError'
        // JAVA_TOOL_OPTIONS = '-Djava.locale.providers=COMPAT,CLDR,SPI --add-opens=java.base/java.lang=ALL-UNNAMED'

        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
        
        // SOLR_OPTS = "${env.SOLR_OPTS} -Djava.locale.providers=COMPAT,CLDR,SPI"
        
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
                    sh """
                    ./gradlew \
                        compileJava \
                        compileTestJava \
                        --parallel \
                        --continue \
                        -PvalidateLogLevels=false
                """
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
            when {
                expression { !params.SKIP_TESTS }
            }

            parallel {
              
                stage('Core Tests') {
                    steps {
                        dir("src/solr-${env.SOLR_VERSION}") {
                            sh """
                                ./gradlew \
                                    :solr:core:test \
                                    ${env.TEST_OPTS} \
                                    -Dtests.seed=${env.TESTS_SEED} \
                                    ${params.RUN_BEAST_TESTS ? "-Dtests.iters=${params.BEAST_COUNT}" : ''} \
                                    --continue \
                                    -PtestJvmArgs='-Xmx1g'
                               """
                        }
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

      // src/solr-10.0.0/solr/packaging/build/distributions/solr-10.0.0-SNAPSHOT.tgz
      // src/solr-10.0.0/solr/packaging/build/distributions/solr-10.0.0-SNAPSHOT-slim.tgz

          stage('Code Quality') {
            parallel {

                stage('Forbidden APIs') {
                    steps {
                        sh './gradlew forbiddenApisMain forbiddenApisTest --continue'
                    }
                }

                stage('Checkstyle') {
                    steps {
                        sh './gradlew checkstyleMain checkstyleTest --continue'
                    }
                    post {
                        always {
                            recordIssues(
                                tools: [checkStyle(pattern: '**/build/reports/checkstyle/*.xml')],
                                qualityGates: [[threshold: 0, type: 'NEW', unstable: true]]
                            )
                        }
                    }
                }

                stage('Rat License Check') {
                    steps {
                        sh './gradlew rat --continue'
                    }
                }
            }
        }

        stage('Assemble Distributions') {
            steps {
                sh """
                    ./gradlew \
                        :solr:packaging:assembleBin \
                        :solr:packaging:assembleSrc \
                        --parallel
                """
            }
        }
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
