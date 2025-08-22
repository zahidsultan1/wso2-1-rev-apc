def BASE_IMAGE = ''

pipeline {
    agent any

    environment {
        // Base image for builds
        REGISTRY_PREFIX   = "ghcr.io/zahidsultan1/wso2mi-staging"
        DEFAULT_IMAGE     = "wso2mi"
        DEFAULT_IMAGE_TAG = "4.4.0"

        // OpenShift registry details
        OCP_REGISTRY_HOST = "default-route-openshift-image-registry.apps.stgocpv1.jazz.com.pk"
        OCP_NAMESPACE     = "wso2"

        PIPELINE_APP_NAME = "${JOB_NAME}"      // unique app name from job
        IMAGE_TAG         = "${BUILD_NUMBER}"

        // ✅ Image name format = namespace/job:build
        FULL_IMAGE_NAME   = "${OCP_REGISTRY_HOST}/${OCP_NAMESPACE}/${PIPELINE_APP_NAME}:${IMAGE_TAG}"

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
                            BASE_IMAGE = "${OCP_REGISTRY_HOST}/${OCP_NAMESPACE}/${PIPELINE_APP_NAME}:${lastSuccessfulBuildNumber}"
                            echo "Using last successful build image: ${BASE_IMAGE}"
                        } else {
                            BASE_IMAGE = "${REGISTRY_PREFIX}/${DEFAULT_IMAGE}:${DEFAULT_IMAGE_TAG}"
                            echo "No previous successful build found. Using default base image: ${BASE_IMAGE}"
                        }
                    }
                }
            }
        }

        // Parallel Maven builds
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
                // (keep other microservice stages unchanged)
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
                          -t $FULL_IMAGE_NAME \
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
                        podman push --tls-verify=false $FULL_IMAGE_NAME
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

                        # Replace placeholders in deployment.yaml with actual values
                        sed "s|<WS02_APP_NAME>|${PIPELINE_APP_NAME}|g; s|<WS02_APP_TAG>|$IMAGE_TAG|g" mi-config/deployment.yaml > mi-config/deployment-ci.yaml

                        # Annotate deployment for traceability
                        oc -n $OCP_NAMESPACE annotate --overwrite deployment ${PIPELINE_APP_NAME} kubernetes.io/change-cause="Deployed build $BUILD_NUMBER" || true

                        # Apply manifest
                        oc -n $OCP_NAMESPACE apply -f mi-config/deployment-ci.yaml
                    """
                }
            }
        }

        // Cleanup
        stage('Cleanup') {
            steps {
                sh """
                    podman rmi $FULL_IMAGE_NAME || true
                """
            }
        }
    }
}
