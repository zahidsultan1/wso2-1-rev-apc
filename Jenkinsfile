def BASE_IMAGE = ''

pipeline {
    agent any

    environment {
        // Base image (kept same as current Jenkinsfile)
        REGISTRY_PREFIX   = "ghcr.io/zahidsultan1/wso2mi-staging"
        DEFAULT_IMAGE     = "wso2mi"
        DEFAULT_IMAGE_TAG = "4.4.0"

        // OpenShift registry details
        OCP_REGISTRY_HOST = "default-route-openshift-image-registry.apps.stgocpv1.jazz.com.pk" // OpenShift registry host (no namespace)
        OCP_NAMESPACE     = "wso2" // namespace/project in OpenShift

        // Each pipeline app name (default to Jenkins job name)
        PIPELINE_APP_NAME = "${JOB_NAME}"   // e.g. "wso2-rev-1-test"

        IMAGE_NAME        = "mi-services" // base repo folder
        IMAGE_TAG         = "${BUILD_NUMBER}"
        DOCKERFILE_PATH   = "mi-config/Dockerfile"
        BUILD_CONTEXT     = "mi-config"
    }

    stages {
        // Select base image
        stage('Base Image') {
            steps {
                script {
                    if (params.CLEAN_BASE) {
                        BASE_IMAGE = "${REGISTRY_PREFIX}/${DEFAULT_IMAGE}:${DEFAULT_IMAGE_TAG}"
                        echo "Using clean base image: ${BASE_IMAGE}"
                    } else {
                        def lastSuccessfulBuild = currentBuild.previousSuccessfulBuild
                        if (lastSuccessfulBuild) {
                            def lastSuccessfulBuildNumber = lastSuccessfulBuild.number
                            BASE_IMAGE = "${OCP_REGISTRY_HOST}/${OCP_NAMESPACE}/${IMAGE_NAME}:${lastSuccessfulBuildNumber}"
                            echo "Using last successful build image: ${BASE_IMAGE}"
                        } else {
                            BASE_IMAGE = "${REGISTRY_PREFIX}/${DEFAULT_IMAGE}:${DEFAULT_IMAGE_TAG}"
                            echo "No previous successful build found. Using default base image: ${BASE_IMAGE}"
                        }
                    }
                }
            }
        }

        // Parallel Maven builds for all microservices
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

        // Image Build
        stage('Image Build') {
            steps {
                script {
                    sh """
                        podman build \
                          --build-arg BASE_IMAGE=$BASE_IMAGE \
                          -f $DOCKERFILE_PATH \
                          -t $OCP_REGISTRY_HOST/$OCP_NAMESPACE/$IMAGE_NAME/$PIPELINE_APP_NAME:$IMAGE_TAG \
                          $BUILD_CONTEXT                   
                    """
                }
            }
        }

        // Push to OpenShift Registry
        stage('Login & Push to OpenShift Registry') {
            steps {
                withCredentials([string(credentialsId: 'ocp-registry-sa-token', variable: 'OCP_TOKEN')]) {
                    sh """
                        echo \$OCP_TOKEN | podman login $OCP_REGISTRY_HOST --username serviceaccount --password-stdin --tls-verify=false
                        podman push --tls-verify=false $OCP_REGISTRY_HOST/$OCP_NAMESPACE/$IMAGE_NAME/$PIPELINE_APP_NAME:$IMAGE_TAG                       
                    """
                }
            }
        }
        // Deploy latest image to OpenShift
        stage('Deploy to OpenShift') {
            steps {
                withCredentials([string(credentialsId: 'ocp-registry-sa-token', variable: 'OCP_TOKEN')]) {
                    sh """
                        echo \$OCP_TOKEN | oc login --token=\$OCP_TOKEN --server=https://api.stgocpv1.jazz.com.pk:6443 --insecure-skip-tls-verify=true
                        oc project $OCP_NAMESPACE

                        # Replace placeholders in deployment.yaml with actual image values
                        sed 's|<WS02_APP_NAME>|$PIPELINE_APP_NAME|g; s|<WS02_APP_TAG>|$IMAGE_TAG|g' mi-config/deployment.yaml > mi-config/deployment-ci.yaml

                        # Add build number for traceability
                        oc -n $OCP_NAMESPACE annotate --overwrite deployment $PIPELINE_APP_NAME kubernetes.io/change-cause="Deployed build $BUILD_NUMBER" || true

                        # Apply updated manifest
                        oc -n $OCP_NAMESPACE apply -f mi-config/deployment-ci.yaml
                    """
                }
            }
        }

      // Cleanup local image (after successful deploy)
        stage('Cleanup') {
            steps {
                sh """
                    podman rmi $OCP_REGISTRY_HOST/$OCP_NAMESPACE/$IMAGE_NAME/$PIPELINE_APP_NAME:$IMAGE_TAG || true
                """
            }
        }

    }
}
