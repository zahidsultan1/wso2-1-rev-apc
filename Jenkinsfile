def BASE_IMAGE = ''

pipeline {
    agent any

    environment {
        REGISTRY_PREFIX = "ghcr.io/zahidsultan1/wso2mi-staging"
        DEFAULT_IMAGE   = "wso2mi"
        DEFAULT_IMAGE_TAG = "4.4.0"

        IMAGE_NAME = "mi-services"
        REGISTRY   = "ghcr.io/zahidsultan1/wso2-mi-services"
        IMAGE_TAG  = "${BUILD_NUMBER}"   // auto-increment tag
        DOCKERFILE_PATH = "mi-config/Dockerfile"
        BUILD_CONTEXT   = "mi-config"
    }

    stages {
        // select base image
        stage('Base Image') {
            steps {
                script {
                    if (params.CLEAN_BASE) {
                        BASE_IMAGE = "${REGISTRY_PREFIX}/${DEFAULT_IMAGE}:${DEFAULT_IMAGE_TAG}"
                        echo "Using clean base image: ${BASE_IMAGE}"
                        echo "WARN: This will include only the selected code"
                    } else {
                        def lastSuccessfulBuild = currentBuild.previousSuccessfulBuild
                        if (lastSuccessfulBuild) {
                            def lastSuccessfulBuildNumber = lastSuccessfulBuild.number
                            BASE_IMAGE = "${REGISTRY}/${IMAGE_NAME}:${lastSuccessfulBuildNumber}"
                            echo "Using last successful build image: ${BASE_IMAGE}"
                        } else {
                            BASE_IMAGE = "${REGISTRY_PREFIX}/${DEFAULT_IMAGE}:${DEFAULT_IMAGE_TAG}"
                            echo "No previous successful build found. Using default base image: ${BASE_IMAGE}"
                        }
                    }
                }
            }
        }

        // create car files (your parallel API builds)
        stage('Parallel') {
            parallel {
                stage('JC Bundle Sub') {
                    when { expression { params.BUILD_JCWallet == null || params.BUILD_JCWallet } }
                    steps {
                        sh '''
                            cd bundlesub-jcwallet
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('JazzRed MsisdnInfo') {
                    when { expression { params.BUILD_JRed_MSISDNinfo == null || params.BUILD_JRed_MSISDNinfo } }
                    steps {
                        sh '''
                            cd jazzred-msisdninfo
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('JazzRed Billdetails') {
                    when { expression { params.BUILD_JRed_BillDetails == null || params.BUILD_JRed_BillDetails } }
                    steps {
                        sh '''
                            cd jazzred-billdetails
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('JazzRed LoanCheck') {
                    when { expression { params.BUILD_JRed_LoanCheck == null || params.BUILD_JRed_LoanCheck } }
                    steps {
                        sh '''
                            cd jazzred-loancheck
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('APC Posting') {
                    when { expression { params.BUILD_APC_Posting == null || params.BUILD_APC_Posting } }
                    steps {
                        sh '''
                            cd apc-posting
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('APC V1') {
                    when { expression { params.BUILD_APC_v1 == null || params.BUILD_APC_v1 } }
                    steps {
                        sh '''
                            cd apc-v1
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('APC V2') {
                    when { expression { params.BUILD_APC_v2 == null || params.BUILD_APC_v2 } }
                    steps {
                        sh '''
                            cd apc-v2
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('ECare APC V3') {
                    when { expression { params.BUILD_APC_v3 == null || params.BUILD_APC_v3 } }
                    steps {
                        sh '''
                            cd apc-v3
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('ECare ChannelAPI V1') {
                    when { expression { params.BUILD_ChannelAPI_v1 == null || params.BUILD_ChannelAPI_v1 } }
                    steps {
                        sh '''
                            cd channelapi-v1
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('ECare ChannelAPI V2') {
                    when { expression { params.BUILD_ChannelAPI_v2 == null || params.BUILD_ChannelAPI_v2 } }
                    steps {
                        sh '''
                            cd channelapi-v2
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('ECare ChannelAPI V3') {
                    when { expression { params.BUILD_ChannelAPI_v3 == null || params.BUILD_ChannelAPI_v3 } }
                    steps {
                        sh '''
                            cd channelapi-v3
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }

                stage('ECare HBL_EasyPaisa') {
                    when { expression { params.BUILD_HBL_EasyPaisa == null || params.BUILD_HBL_EasyPaisa } }
                    steps {
                        sh '''
                            cd hbl-easypaisa
                            mvn -e clean install -Dmaven.test.skip=true
                        '''
                    }
                }
            }
        }

        // build & push final image with Podman
        stage('Build & Push') {
            steps {
                script {
                    sh """
                        podman build \
                          --build-arg BASE_IMAGE=$BASE_IMAGE \
                          -f $DOCKERFILE_PATH \
                          -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG \
                          $BUILD_CONTEXT
                    """

                    withCredentials([usernamePassword(credentialsId: 'ghcr-creds', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_TOKEN')]) {
                        sh """
                            echo $GITHUB_TOKEN | podman login ghcr.io -u $GITHUB_USER --password-stdin
                            podman push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                            podman rmi  $REGISTRY/$IMAGE_NAME:$IMAGE_TAG || true
                        """
                    }
                }
            }
        }
    }
}

