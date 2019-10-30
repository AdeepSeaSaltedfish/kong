pipeline {
    agent none
    environment{
        KONG_PACKAGE_NAME = 'kong'
        REPOSITORY_NAME = 'kong-nightly'
        REPOSITORY_OS_NAME = "${env.BRANCH_NAME}"
        UPDATE_CACHE = "true"
        DOCKER_CREDENTIALS = credentials('dockerhub')
        DOCKER_USERNAME = "${env.DOCKER_CREDENTIALS_USR}"
        DOCKER_PASSWORD = "${env.DOCKER_CREDENTIALS_PSW}"
        BINTRAY_CREDENTIALS = credentials('bintray-ce')
        BINTRAY_USR = "${env.BINTRAY_CREDENTIALS_USR}"
        BINTRAY_KEY = "${env.BINTRAY_CREDENTIALS_PSW}"
        KONG_BUILD_TOOLS = "origin/feat/kong-jenkins"
    }    
    stages {
        stage('Build Kong') {
            agent {
                node {
                    label 'docker-compose'
                }
            }
            environment {
                KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
            }
            steps {
                sh 'make setup-kong-build-tools'
                sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                dir('../kong-build-tools') { sh 'make kong-test-container' }
            }
        }
        stage('Integration Tests') {
            parallel {
                stage('dbless') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                        TEST_DATABASE = "off"
                        TEST_SUITE = "dbless"
                    }
                    steps {
                        sh 'make setup-kong-build-tools'
                        dir('../kong-build-tools'){
                            sh 'make test-kong'
                        }
                    }
                }
                stage('postgres') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                        TEST_DATABASE = 'postgres'
                    }
                    steps {
                        sh 'make setup-kong-build-tools'
                        dir('../kong-build-tools'){
                            sh 'make test-kong'
                        }
                    }
                }
                stage('postgres plugins') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                        TEST_DATABASE = 'postgres'
                        TEST_SUITE = 'plugins'
                    }
                    steps {
                        sh 'make setup-kong-build-tools'
                        dir('../kong-build-tools'){
                            sh 'make test-kong'
                        }
                    }
                }
                stage('cassandra') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                        TEST_DATABASE = 'cassandra'
                    }
                    steps {
                        sh 'make setup-kong-build-tools'
                        dir('../kong-build-tools'){
                            sh 'make test-kong'
                        }
                    }
                }
            }
        }
        stage('Nightly Releases') {
            parallel {
                stage('Ubuntu Releases') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = 'deb'
                        RESTY_IMAGE_BASE = 'ubuntu'
                        RESTY_IMAGE_TAG = 'xenial'
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                    }
                    steps {
                        sh 'make setup-kong-build-tools'
                        sh 'mkdir -p $HOME/bin'
                        sh 'sudo ln -s $HOME/bin/kubectl /usr/local/bin/kubectl'
                        sh 'sudo ln -s $HOME/bin/kind /usr/local/bin/kind'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'export KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=trusty && make nightly-release'
                        sh 'export KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=xenial && make nightly-release'
                        sh 'export KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=bionic && make nightly-release'
                    }
                }
                stage('Centos Releases') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = 'rpm'
                        RESTY_IMAGE_BASE = 'centos'
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                        REDHAT_CREDENTIALS = credentials('redhat')
                        REDHAT_USERNAME = "${env.REDHAT_USR}"
                        REDHAT_PASSWORD = "${env.REDHAT_PSW}"
                    }
                    steps {
                        sh 'make setup-kong-build-tools'
                        sh 'mkdir -p $HOME/bin'
                        sh 'sudo ln -s $HOME/bin/kubectl /usr/local/bin/kubectl'
                        sh 'sudo ln -s $HOME/bin/kind /usr/local/bin/kind'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'export KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=6 && make nightly-release'
                        sh 'export KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=7 && make nightly-release'
                    }
                }
                stage('Debian Releases') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = 'deb'
                        RESTY_IMAGE_BASE = 'debian'
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                    }
                    steps {
                        sh 'make setup-kong-build-tools'
                        sh 'mkdir -p $HOME/bin'
                        sh 'sudo ln -s $HOME/bin/kubectl /usr/local/bin/kubectl'
                        sh 'sudo ln -s $HOME/bin/kind /usr/local/bin/kind'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'export KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=jessie && make nightly-release'
                        sh 'export KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=stretch && make nightly-release'
                        sh 'export KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=buster && make nightly-release'
                    }
                }
            }
        }
    }
}