def BASE_IMAGE = ''


pipeline {
    agent {
        kubernetes {
            yamlFile 'builder.yaml'
        }
    }

    environment {
        REGISTRY_PREFIX = "internal-registry.tkg.mobilink.net.pk/wso2-prod/wso2"
        DEFAULT_IMAGE = "wso2mi"
        DEFAULT_IMAGE_TAG = "4.1.0-ol7"
        GATEWAY_FLAG = "FALSE"
    }

    stages {
        // select base image
        stage('Base Image') {
            steps {
                script {
                    if (params.CLEAN_BASE){
                        BASE_IMAGE = "${REGISTRY_PREFIX}/${DEFAULT_IMAGE}:${DEFAULT_IMAGE_TAG}"
                        echo "Using clean base image: ${BASE_IMAGE}"
                        echo "WARN: This will include only the selected code"
                    }else{
                        def lastSuccessfulBuild = currentBuild.previousSuccessfulBuild
                        if (lastSuccessfulBuild) {
                            def lastSuccessfulBuildNumber = lastSuccessfulBuild.number
                            BASE_IMAGE = "${REGISTRY_PREFIX}/mi-services/${JOB_NAME}:${lastSuccessfulBuildNumber}"
                            echo "Using last successful build image: ${BASE_IMAGE}"
                        } else {
                            BASE_IMAGE = "${REGISTRY_PREFIX}/${DEFAULT_IMAGE}:${DEFAULT_IMAGE_TAG}"
                            echo "No previous successful build found. Using default base image: ${BASE_IMAGE}"
                        }
                    }
                    
                }
            }
        }

        // create car files
         stage('Parallel') {
            parallel {
                stage('JC Bundle Sub') {
                    when {
                        expression { params.BUILD_JCWallet == null || params.BUILD_JCWallet }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd bundlesub-jcwallet
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('JazzRed MsisdnInfo') {
                    when {
                        expression { params.BUILD_JRed_MSISDNinfo == null || params.BUILD_JRed_MSISDNinfo }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd jazzred-msisdninfo
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('JazzRed Billdetails') {
                    when {
                        expression { params.BUILD_JRed_BillDetails == null || params.BUILD_JRed_BillDetails }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd jazzred-billdetails
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('JazzRed LoanCheck') {
                    when {
                        expression { params.BUILD_JRed_LoanCheck == null || params.BUILD_JRed_LoanCheck }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd jazzred-loancheck
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('APC Posting') {
                    when {
                        expression { params.BUILD_APC_Posting == null || params.BUILD_APC_Posting }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd apc-posting
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('APC V1') {
                    when {
                        expression { params.BUILD_APC_v1 == null || params.BUILD_APC_v1 }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd apc-v1
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('APC V2') {
                    when {
                        expression { params.BUILD_APC_v2== null || params.BUILD_APC_v2 }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd apc-v2
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('ECare APC V3') {
                    when {
                        expression { params.BUILD_APC_v3 == null || params.BUILD_APC_v3 }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd apc-v3
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('ECare ChannelAPI V1') {
                    when {
                        expression { params.BUILD_ChannelAPI_v1 == null || params.BUILD_ChannelAPI_v1 }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd channelapi-v1
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('ECare ChannelAPI V2') {
                    when {
                        expression { params.BUILD_ChannelAPI_v2 == null || params.BUILD_ChannelAPI_v2 }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd channelapi-v2
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('ECare ChannelAPI V3') {
                    when {
                        expression { params.BUILD_ChannelAPI_v3 == null || params.BUILD_ChannelAPI_v3 }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd channelapi-v3
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }
				
				stage('ECare HBL_EasyPaisa') {
                    when {
                        expression { params.BUILD_HBL_EasyPaisa == null || params.BUILD_HBL_EasyPaisa }
                    }
                    steps {
                        container('maven') {
                            script {
                            sh '''
                                cd hbl-easypaisa
                                mvn -e clean install -Dmaven.test.skip=true
                                '''
                            }
                        }
                    }
                }								

            }
         }
        
        // container image build and push
        stage('Build & Push') {
            steps {
                container('kaniko') {
                    script {
                        env.BASE_IMAGE = BASE_IMAGE

                        sh '''
                        cd mi-config
                        /kaniko/executor --cleanup --dockerfile `pwd`/Dockerfile \
                                         --context `pwd` \
                                         --destination=internal-registry.tkg.mobilink.net.pk/wso2-prod/wso2/mi-services/$JOB_NAME:$BUILD_NUMBER \
                                         --build-arg BASE_IMAGE=$BASE_IMAGE
                        '''
                    }
                }
            }
        }

        // deploy
        stage('Deploy') {
            steps 
            {
                container('kubectl') {
                    withKubeCredentials(kubectlCredentials: [[caCertificate: '''MIICyzCCAbOgAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
                        cm5ldGVzMB4XDTIyMDgwMzA5MzQyOVoXDTMyMDczMTA5MzkyOVowFTETMBEGA1UE
                        AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAM5a
                        U28HQM5Gbdbb/r/GpkHeTW857o7Hxo19LSIerdoMzPaC0mTPktn4cEOgbE97BrEU
                        f7Y9Cmt6o4a7B9HGlPO9ID7sZ3RBJFLwXNeHq1MJCMwGiOTn2C08Qtrf2pqGpovm
                        VO96XyzgPCFqyNoklpiPsZ66iX/ajfAP0oXbP92mSqhVXFAE/DzWZQ792QjuKY3a
                        AsjJXi/nYFRNToh0uAGcr70IbuoMygJZzUTW4kTXfsuaTAYQxBs2d2cTn4bUGDe0
                        yOtQ2dAJLoTg/901OquQpgNLPBTNNpnm/aVNj6jOjpMS512XOw2RqgjBtLBUO1qP
                        XJ0GFbS1bMhXMcaP+4MCAwEAAaMmMCQwDgYDVR0PAQH/BAQDAgKkMBIGA1UdEwEB
                        /wQIMAYBAf8CAQAwDQYJKoZIhvcNAQELBQADggEBAG0isEO70igrP4v165MsRI73
                        sA6D7ZFDZ9GSF9brsmeLsUID0AD33pU7HDD8KTsGvrg+ZE4I/kgZZHxKzjc2LoAj
                        41VyL95wUwUQAQNZVPPjXc3Nfzow2FLG6J3BXFBg3VIMr3nuydhdLWIJz71T9JGK
                        mDGeGn1hw0yR4vxC3P/PJYyWI8xp8wovr+323jIVBZAUNRz8rVAwK3cjfIqRumEi
                        V6VoMMMvN0wUc1Mlyjvl//GTjVrndQ4KXLly11cNphHRQDwux21uzJ+hFa6LM5xg
                        jNUrsA2GfXKitOevaBFGYVXqbY/pP2e0Hsn89JC7Tzf8AVCihyGE3dlo1beRWBs=''', 
                        clusterName: '10.50.49.24', 
                        contextName: 'tkc-prod-wso2-ns001', 
                        credentialsId: 'jenkins-kubectl', 
                        namespace: 'jenkins', serverUrl: 'https://10.50.49.24:6443']]) {
                        sh '''
                            cd mi-config
                            sed -e "s/<WS02_APP_NAME>/\${JOB_NAME}/g" -e "s/<WS02_APP_TAG>/\${BUILD_NUMBER}/g" deployment.yaml > deployment-final.yaml
                            kubectl apply -f deployment-final.yaml
                        '''
                    }
                }
            }
        }
        stage('Publish') {
          when {
            expression {
              return env.GATEWAY_FLAG == 'TRUE';
            }
          }
          steps {
            container('kaniko') {
              script {
                sh '''
                cd gw-config
                sed -e "s/<WS02_APP_NAME>/\${JOB_NAME}/g" -e "s/<WS02_APP_TAG>/\${BUILD_NUMBER}/g" deployment.toml > deployment-final.toml
                /kaniko/executor --cleanup --dockerfile `pwd`/Dockerfile \
                                  --context `pwd` \
                                  --destination=internal-registry.tkg.mobilink.net.pk/wso2-prod/wso2/wso2gw/$JOB_NAME:$BUILD_NUMBER
                '''
              }
            }
            container('kubectl') {
              withKubeCredentials(kubectlCredentials: [[caCertificate: '''MIICyzCCAbOgAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
                cm5ldGVzMB4XDTIyMDgwMzA5MzQyOVoXDTMyMDczMTA5MzkyOVowFTETMBEGA1UE
                AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAM5a
                U28HQM5Gbdbb/r/GpkHeTW857o7Hxo19LSIerdoMzPaC0mTPktn4cEOgbE97BrEU
                f7Y9Cmt6o4a7B9HGlPO9ID7sZ3RBJFLwXNeHq1MJCMwGiOTn2C08Qtrf2pqGpovm
                VO96XyzgPCFqyNoklpiPsZ66iX/ajfAP0oXbP92mSqhVXFAE/DzWZQ792QjuKY3a
                AsjJXi/nYFRNToh0uAGcr70IbuoMygJZzUTW4kTXfsuaTAYQxBs2d2cTn4bUGDe0
                yOtQ2dAJLoTg/901OquQpgNLPBTNNpnm/aVNj6jOjpMS512XOw2RqgjBtLBUO1qP
                XJ0GFbS1bMhXMcaP+4MCAwEAAaMmMCQwDgYDVR0PAQH/BAQDAgKkMBIGA1UdEwEB
                /wQIMAYBAf8CAQAwDQYJKoZIhvcNAQELBQADggEBAG0isEO70igrP4v165MsRI73
                sA6D7ZFDZ9GSF9brsmeLsUID0AD33pU7HDD8KTsGvrg+ZE4I/kgZZHxKzjc2LoAj
                41VyL95wUwUQAQNZVPPjXc3Nfzow2FLG6J3BXFBg3VIMr3nuydhdLWIJz71T9JGK
                mDGeGn1hw0yR4vxC3P/PJYyWI8xp8wovr+323jIVBZAUNRz8rVAwK3cjfIqRumEi
                V6VoMMMvN0wUc1Mlyjvl//GTjVrndQ4KXLly11cNphHRQDwux21uzJ+hFa6LM5xg
                jNUrsA2GfXKitOevaBFGYVXqbY/pP2e0Hsn89JC7Tzf8AVCihyGE3dlo1beRWBs=''', 
                clusterName: '10.50.49.24', 
                contextName: 'tkc-prod-wso2-ns001', 
                credentialsId: 'jenkins-kubectl', 
                namespace: 'jenkins', serverUrl: 'https://10.50.49.24:6443']]) {
                      sh '''
                        cd gw-config
                        sed -e "s/<WS02_APP_NAME>/\${JOB_NAME}/g" -e "s/<WS02_APP_TAG>/\${BUILD_NUMBER}/g" deployment.yaml > deployment-final.yaml
                        kubectl apply -f deployment-final.yaml
                      '''
                  }
                }
            } 
        }        
    }
}
