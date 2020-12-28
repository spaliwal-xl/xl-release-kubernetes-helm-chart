pipeline {
    agent any
    parameters {
    choice(name: 'PRODUCT', choices: ['XL Release', 'XL Deploy'], description: 'Select the product to package')
    choice(name: 'PLATFORM', choices: ['Onprem', 'EKS', 'Openshift_AWS','Openshift_Onprem'], description: 'Pick the platform to deploy the helm chart')
    choice(name: 'INSTALL_CHART', choices: ['YES', 'NO'], description: 'Do you want to install the helm chart?')
  }

  options {
        timeout(time: 1, unit: 'HOURS')
    }

  environment {
        BRANCH_NAME = "master"
        PRODUCT_NAME = "${params.PRODUCT}"
        GIT_HUB_XLR_URL = "https://github.com/chandras-xl/xl-release-kubernetes-helm-chart.git"
        GIT_HUB_XLD_URL = "https://github.com/chandras-xl/xl-deploy-kubernetes-helm-chart-1.git"
        NEXUS_USER = "admin"
        NEXUS_PASSWORD = credentials('NEXUS_SECRET')
        NEXUS_URL = "http://nexus-appserver.digitalai-testing.com:8081/repository/helm_repository/"
    }

  stages {
        stage('Git checkout') {
            steps {
                script {
                  if ( env.BRANCH_NAME == 'master' && env.PRODUCT == 'XL Release' ){
                    try{
                      echo "checking out the code for ${params.PRODUCT} from ${env.BRANCH_NAME} branch"
                      sh "git clone -b ${env.BRANCH_NAME} ${env.GIT_HUB_XLR_URL}"
                      echo "Git checkout successful"
                    }catch (error){
                      echo "checkout failed"
                      throw error
                      }
                  } else {
                  	try{
                  	  echo "checking out the code for ${params.PRODUCT} from ${env.BRANCH_NAME} branch"
                  	  sh "git clone -b ${env.BRANCH_NAME} ${env.GIT_HUB_XLD_URL}"
                      echo "Git checkout successful"
                  	}catch(error){
                  	  echo "checkout failed"
                      throw(error)
                  	}
                  }
                }
            }
        }
        stage('Packaging helm') {
          steps {
            script {
              if ( env.PRODUCT_NAME == 'XL Release' ){
              try{
                echo "Updating the helm dependencies"
                  sh "/usr/local/bin/helm dependency update xl-release-kubernetes-helm-chart "
                echo "Packaging the helm chart ${params.PRODUCT}"
                sh "/usr/local/bin/helm package xl-release-kubernetes-helm-chart "
                echo "Build package completed"
              }catch(error){
                echo "Packing failed"
                throw error
                  }
              } else {
                echo "packaging helm chart ${params.PRODUCT} "
               try{
                echo "Updating the helm dependencies"
                sh "/usr/local/bin/helm dependency update xl-deploy-kubernetes-helm-chart-1 "
                echo "Packing the helm chart ${params.PRODUCT}"
                sh "/usr/local/bin/helm package xl-deploy-kubernetes-helm-chart-1 "
                echo "Build package completed"
                }catch(error){
                  echo "Packaging failed"
                  throw error
                 }
              }
            }
          }
        }
        stage('push to nexus'){
          steps {
               sh "echo pushing the build to nexus"
               script {
                   if ( env.PRODUCT_NAME == 'XL Release' ){
                   try{
                      sh "curl -u ${NEXUS_USER}:${NEXUS_PASSWORD} ${NEXUS_URL} --upload-file *release*.tgz -v"
                      echo "push completed"
                    }catch(error){
                      echo "push to nexus failed"
                      throw error
                        }
                   } else {
                     try{
                      sh "curl -u ${NEXUS_USER}:${NEXUS_PASSWORD} ${NEXUS_URL} --upload-file *deploy*.tgz -v"
                        echo "push completed"
                     } catch(error){
                      echo "push to nexus failed"
                      throw error
                     }
                   }
               }
          }
        }
        stage('push to xebialabs dist'){
          steps {
             script {
                 if ( env.PRODUCT_NAME == 'XL Release' ){
                  try{
                    echo "pushing ${params.PRODUCT} zip file to xebialabs distribution site"
                  }catch(error){
                    echo "push failed"
                    throw error
                      }
                 }else {
                  try{
                    echo "pushing ${params.PRODUCT} zip file to xebialabs distribution site"
                  }catch(error){
                    echo "push failed"
                    throw error
                      }
                 }
             }
          }
        }
        stage('Deploy to Onprem cluster'){
          when {
            anyOf{
              expression{ params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Release' && params.PLATFORM == 'Onprem' }
              expression{ params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Deploy' && params.PLATFORM == 'Onprem' }
            }
          }
          steps {
               script {
                   withCredentials([usernamePassword(credentialsId: 'ON_PREM_K8S', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                   withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: 'kubernetes-admin@kubernetes', credentialsId: 'ON_PREM_K8S', namespace: 'default', serverUrl: 'https://172.16.16.21:6443') {
                   load "$JENKINS_HOME/.env/param.groovy"
                   if ( env.PRODUCT == 'XL Release') {
                    try {
                       echo "Installing the ${params.PRODUCT} chart"
                       withCredentials([string(credentialsId: 'XLD_LIC', variable: 'XLR_LICENSE')]) {
                       withCredentials([string(credentialsId: 'XLR_KEYSTORE', variable: 'XLR_KEYSTORE')]) {
                       withCredentials([string(credentialsId: 'XLR_PASS_PHRASE', variable: 'XLR_PASS_PHRASE')]) {
                       sh "/usr/local/bin/helm install --generate-name *.tgz --set ingress.hosts[0]=${HostName_XLR_onprem} --set xlrLicense=${XLR_LICENSE} --set RepositoryKeystore=${XLR_KEYSTORE} --set KeystorePassphrase=${XLR_PASS_PHRASE} --set Persistence.StorageClass=${StorageClass_onprem}"
                       sh "sleep 5"
                       sh "/usr/local/bin/kubectl get svc"
                       echo "================================================================"
                       echo "${params.PRODUCT} deployed successful on ${params.PLATFORM}!!!!"
                       echo "================================================================"
                     }
                     }
                     }
                   }catch (error){
                     echo "Deployment failed"
                     throw error
                     }
                   }else {
                     try {
                       echo "Installing the ${params.PRODUCT} chart"
                       withCredentials([string(credentialsId: 'XLD_LIC', variable: 'XLD_LICENSE')]) {
                       withCredentials([string(credentialsId: 'XLD_KEYSTORE', variable: 'XLD_KEYSTORE')]) {
                       withCredentials([string(credentialsId: 'XLD_PASS_PHRASE', variable: 'XLD_PASS_PHRASE')]) {
                       sh "/usr/local/bin/helm install --generate-name *.tgz --set ingress.hosts[0]=${HostName_XLD_onprem} --set xldLicense=${XLD_LICENSE} --set RepositoryKeystore=${XLD_KEYSTORE} --set KeystorePassphrase=${XLD_PASS_PHRASE} --set Persistence.StorageClass=${StorageClass_onprem}"
                       sh "sleep 5"
                       sh "/usr/local/bin/kubectl get svc"
                       echo "================================================================"
                       echo "${params.PRODUCT} deployed successful on ${params.PLATFORM}!!!!"
                       echo "================================================================"
                     }
                     }
                     }
                   }catch (error){
                       echo "Deployment failed"
                      throw error
                    }
                   }
                   }
                   }

               }
          }
        }
        stage('Deploy to EKS cluster'){
          when {
            anyOf{
              expression{ params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Release' && params.PLATFORM == 'EKS' }
              expression{ params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Deploy' && params.PLATFORM == 'EKS'}
            }
          }
          steps {
               script {
                   withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS_CRED', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                   withKubeConfig(caCertificate: '', clusterName: 'eks-devops', contextName: 'spaliwal@eks-devops.eu-west-1.eksctl.io', credentialsId: 'EKS_CONFIG', namespace: 'default', serverUrl: 'https://CC5E2351F79187EF9D05C18AA2FE52D7.gr7.eu-west-1.eks.amazonaws.com') {
                   load "$JENKINS_HOME/.env/param.groovy"
                   if ( env.PRODUCT == 'XL Release') {
                    try {
                       echo "Installing the ${params.PRODUCT} chart"
                       withCredentials([string(credentialsId: 'XLR_LIC', variable: 'XLR_LICENSE')]) {
                       withCredentials([string(credentialsId: 'XLR_KEYSTORE', variable: 'XLR_KEYSTORE')]) {
                       withCredentials([string(credentialsId: 'XLR_PASS_PHRASE', variable: 'XLR_PASS_PHRASE')]) {
                       sh "/usr/local/bin/helm install --generate-name *.tgz --set ingress.hosts[0]=${HostName_XLR} --set haproxy-ingress.controller.service.type=${LOADBALANCER} --set xlrLicense=${XLR_LICENSE} --set RepositoryKeystore=${XLR_KEYSTORE} --set KeystorePassphrase=${XLR_PASS_PHRASE} --set Persistence.StorageClass=${StorageClass}"
                       sh "sleep 5"
                       sh "/usr/local/bin/kubectl get svc"
                       echo "================================================================"
                       echo "${params.PRODUCT} deployed successful on ${params.PLATFORM}!!!!"
                       echo "================================================================"
                     }
                     }
                     }
                   }catch (error){
                     echo "Deployment failed"
                     throw error
                     }
                   }else {
                     try {
                       echo "Installing the ${params.PRODUCT} chart"
                       withCredentials([string(credentialsId: 'XLD_LIC', variable: 'XLD_LICENSE')]) {
                       withCredentials([string(credentialsId: 'XLD_KEYSTORE', variable: 'XLD_KEYSTORE')]) {
                       withCredentials([string(credentialsId: 'XLD_PASS_PHRASE', variable: 'XLD_PASS_PHRASE')]) {
                       sh "/usr/local/bin/helm install --generate-name *.tgz --set ingress.hosts[0]=${HostName_XLD} --set haproxy-ingress.controller.service.type=${LOADBALANCER} --set xldLicense=${XLD_LICENSE} --set RepositoryKeystore=${XLD_KEYSTORE} --set KeystorePassphrase=${XLD_PASS_PHRASE} --set Persistence.StorageClass=${StorageClass}"
                       sh "sleep 5"
                       sh "/usr/local/bin/kubectl get svc"
                       echo "================================================================"
                       echo "${params.PRODUCT} deployed successful on ${params.PLATFORM}!!!!"
                       echo "================================================================"
                     }
                     }
                     }
                   }catch (error){
                       echo "Deployment failed"
                      throw error
                    }
                   }
                   }
                   }

               }
          }
        }
   }
   post {
        always {
            cleanWs()
        }
    }
}
