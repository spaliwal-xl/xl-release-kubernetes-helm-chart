pipeline{
    agent any
    parameters {
        choice(name: 'PRODUCT', choices: ['XL Release', 'XL Deploy'], description: 'Select the product to package')
        choice(name: 'PLATFORM', choices: ['Onprem', 'EKS', 'Openshift_AWS','Openshift_Onprem'], description: 'Pick the platform to deploy the helm chart')
        choice(name: 'INSTALL_CHART', choices: ['YES', 'NO'], description: 'Do you want to install the helm chart?')
    }
    options {
        timeout(time: 10, unit: 'MINUTES')
    }
    environment {
        BRANCH_NAME = "rabbitmq"
        BRANCH_OPENSHIFT = "ENG-3449"
        PRODUCT_NAME = "${params.PRODUCT}"
        PLATFORM_NAME = "${params.PLATFORM}"
        GIT_HUB_XLR_URL = "https://github.com/chandras-xl/xl-release-kubernetes-helm-chart.git"
        GIT_HUB_XLD_URL = "https://github.com/xebialabs/xl-deploy-kubernetes-helm-chart.git"
        GIT_HUB_XLR_OPENSHIFT_URL = "https://github.com/xebialabs/xl-release-kubernetes-helm-chart.git"
        GIT_HUB_XLD_OPENSHIFT_URL = "https://github.com/xebialabs/xl-deploy-kubernetes-helm-chart.git"       
        NEXUS_USER = "admin"
        NEXUS_PASSWORD = credentials('NEXUS_SECRET')
        NEXUS_URL_XLR = "http://nexus-appserver.digitalai-testing.com:8081/repository/XL_Release_Helm_Repo/"
        NEXUS_URL_XLD = "http://nexus-appserver.digitalai-testing.com:8081/repository/XL_Deploy_Helm_Repo/"
        NEXUS_URL_OC_XLR = "http://nexus-appserver.digitalai-testing.com:8081/repository/XL_Release_Openshift_Helm_Repo/"
        NEXUS_URL_OC_XLD = "http://nexus-appserver.digitalai-testing.com:8081/repository/XL_Deploy_Openshift_Helm_Repo/"
        XEBIALABS_DIST_USERNAME = "download"
        XEBIALABS_DIST_PASSWORD = ""
        XEBIALABS_DIST_XLR_URL = ""
        XEBIALABS_DIST_XLD_URL = ""
        XEBIALABS_DIST_OC_XLR_URL = ""
        XEBIALABS_DIST_OC_XLD_URL = ""
        HOST_NAME_XLR_ONPREM = "kublove-kn2.xebialabs.com"
        HOST_NAME_XLD_ONPREM = "kublove-kn3.xebialabs.com"
        HOST_NAME_XLR_EKS = "release-eks.digitalai-testing.com"      
        HOST_NAME_XLD_EKS = "deploy-eks.digitalai-testing.com"
        HOST_NAME_XLR_OPENSHIFT = "release.apps.ocp-qa.xldevinfra.com"
        HOST_NAME_XLD_OPENSHIFT = "deploy.apps.ocp-qa.xldevinfra.com"
        STORAGE_CLASS_ONPREM = "nfs-client"
        STORAGE_CLASS_EKS = "aws-efs"
        LOADBALANCER = "LoadBalancer"
        OPENSHIFT_AWS_CRED = credentials('OPENSHIFT_AWS_CRED')
        OPENSHIFT_AWS_SERVER_URL = "https://api.ocp-qa.xldevinfra.com:6443"
    }
    stages {
        stage('Git checkout') {
            steps {
                script {
                    if ( env.PRODUCT_NAME == 'XL Release' && (env.PLATFORM_NAME == 'Onprem' | env.PLATFORM_NAME == 'EKS' )) {
                        try {
                            sh '''
                                echo Checking out the code for $PRODUCT_NAME $PLATFORM_NAME platform from $BRANCH_NAME branch
                                git clone -b $BRANCH_NAME $GIT_HUB_XLR_URL
                                echo "Git check out successful !!!"
                            '''
                        }catch(error) {
                            throw error
                        }
                    } else if ( env.PRODUCT_NAME == 'XL Deploy' && (env.PLATFORM_NAME == 'Onprem' | env.PLATFORM_NAME == 'EKS' )) {
                        try {
                            sh '''
                                echo Checking out the code for $PRODUCT_NAME $PLATFORM_NAME platform from $BRANCH_NAME branch
                                git clone -b $BRANCH_NAME $GIT_HUB_XLD_URL
                                echo "Git check out successful !!!"
                            '''
                        }catch(error) {
                            throw error
                        }
                    } else if ( env.PRODUCT_NAME == 'XL Release' && (env.PLATFORM_NAME == 'Openshift_Onprem' | env.PLATFORM_NAME == 'Openshift_AWS' )) {
                        try {
                            sh '''
                                echo Checking out the code for $PRODUCT_NAME $PLATFORM_NAME platform from $BRANCH_NAME branch
                                git clone -b $BRANCH_OPENSHIFT $GIT_HUB_XLR_OPENSHIFT_URL
                                echo "Git check out successful !!!"
                            '''
                        }catch(error) {
                            throw error
                        }
                    } else if ( env.PRODUCT_NAME == 'XL Deploy' && (env.PLATFORM_NAME == 'Openshift_Onprem' | env.PLATFORM_NAME == 'Openshift_AWS' )) {
                        try {
                            sh '''
                                echo Checking out the code for $PRODUCT_NAME $PLATFORM_NAME platform from $BRANCH_NAME branch
                                git clone -b $BRANCH_OPENSHIFT $GIT_HUB_XLD_OPENSHIFT_URL
                                echo "Git check out successful !!!"
                            '''
                        }catch(error) {
                            throw error
                        }
                    } else {
                        echo "checkout failed !!!"
                    }
                }
            }
        }
        stage('Package helm') {
            steps {
                script {
                    if ( env.PRODUCT_NAME == 'XL Release' ) {
                        try {
                            sh '''
                                echo Updating the helm dependencies for $PRODUCT_NAME
                                /usr/local/bin/helm dependency update xl-release-kubernetes-helm-chart
                                echo Packing the helm chart $PRODUCT_NAME
                                /usr/local/bin/helm package xl-release-kubernetes-helm-chart
                                echo Build package completed !!!
                            '''
                        }catch(error) {
                            throw error
                        }
                    } else {
                        try {
                            sh '''
                                echo Updating the helm dependencies for $PRODUCT_NAME
                                /usr/local/bin/helm dependency update xl-deploy-kubernetes-helm-chart
                                echo Packing the helm chart $PRODUCT_NAME
                                /usr/local/bin/helm package xl-deploy-kubernetes-helm-chart
                                echo Build package completed !!!                               
                            '''
                        }catch(error) {
                            throw error
                        }
                    }
                }
            }
        }
        stage('Push to nexus') {
            steps {
                script {
                    if ( env.PRODUCT_NAME == 'XL Release' && (env.PLATFORM_NAME == 'Onprem' | env.PLATFORM_NAME == 'EKS' )) {
                        try {
                            sh '''
                                echo Pushing the $PRODUCT_NAME build to nexus
                                curl -u ${NEXUS_USER}:${NEXUS_PASSWORD} ${NEXUS_URL_XLR} --upload-file *release*.tgz -v
                                echo Pushed $PRODUCT_NAME build successfull to nexus repository ${NEXUS_URL_XLR} 
                            '''
                        }catch(error){
                            throw error
                        }
                    } else if ( env.PRODUCT_NAME == 'XL Deploy' && (env.PLATFORM_NAME == 'EKS' | env.PLATFORM_NAME == 'Onprem' )) {
                        try {
                            sh '''
                                echo Pushing the $PRODUCT_NAME build to nexus
                                curl -u ${NEXUS_USER}:${NEXUS_PASSWORD} ${NEXUS_URL_XLD} --upload-file *deploy*.tgz -v
                                echo Pushed $PRODUCT_NAME build successfull to nexus repository ${NEXUS_URL_XLD}                                 
                            '''
                        }catch(error) {
                            throw error
                        }
                    } else if ( env.PRODUCT_NAME == 'XL Release' && (env.PLATFORM_NAME == 'Openshift_Onprem' | env.PLATFORM_NAME == 'Openshift_AWS' )) {
                        try {
                            sh '''
                                echo Pushing the $PRODUCT_NAME build to nexus
                                curl -u ${NEXUS_USER}:${NEXUS_PASSWORD} ${NEXUS_URL_OC_XLR} --upload-file *release*.tgz -v
                                echo Pushed $PRODUCT_NAME build successfull to nexus repository ${NEXUS_URL_OC_XLR} 
                            '''
                        }catch(error) {
                            throw error
                        }
                    } else if ( env.PRODUCT_NAME == 'XL Deploy' && (env.PLATFORM_NAME == 'Openshift_Onprem' | env.PLATFORM_NAME == 'Openshift_AWS' )) {
                        try {
                            sh '''
                                echo Pushing the $PRODUCT_NAME build to nexus
                                curl -u ${NEXUS_USER}:${NEXUS_PASSWORD} ${NEXUS_URL_OC_XLD} --upload-file *deploy*.tgz -v
                                echo Pushed $PRODUCT_NAME build successfull to nexus repository ${NEXUS_URL_OC_XLD}  
                            '''                           
                        }catch(error) {
                            throw error
                        }
                    } else {
                        echo "Push to nexus failed !!!"
                    }
                }
            }
        }
        stage('Push to Xebialabs dist') {
            steps {
                script {
                    echo "Pushing the build to Xebilabs dist"
                }
            }
        }
        stage('Platforms') {
            parallel {                
                stage('Openshift_AWS') {
                    when {
                        anyOf {
                            expression { params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Release' && params.PLATFORM == 'Openshift_AWS' }
                            expression { params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Deploy' && params.PLATFORM == 'Openshift_AWS' }                      
                        }
                    }
                    steps {
                        script {
                            if ( env.PRODUCT_NAME == 'XL Release' && env.PLATFORM_NAME == 'Openshift_AWS') {
                                try {
                                    withCredentials([string(credentialsId: 'XLR_LIC', variable: 'XLR_LICENSE')]) {
                                        withCredentials([string(credentialsId: 'XLR_KEYSTORE', variable: 'XLR_KEYSTORE')]) {
                                            withCredentials([string(credentialsId: 'XLR_PASS_PHRASE', variable: 'XLR_PASS_PHRASE')]) {
                                                sh '''
                                                    echo "Installing $PRODUCT_NAME on $PLATFORM_NAME platform"
                                                    /usr/local/bin/oc login --username=kubeadmin --password=$OPENSHIFT_AWS_CRED --server=$OPENSHIFT_AWS_SERVER_URL
                                                    /usr/local/bin/oc project pipeline
                                                    /usr/local/bin/helm install --generate-name *.tgz --set route.hosts[0]=$HOST_NAME_XLR_OPENSHIFT --set xlrLicense=${XLR_LICENSE} --set RepositoryKeystore=${XLR_KEYSTORE} --set KeystorePassphrase=${XLR_PASS_PHRASE} --set Persistence.StorageClass=$STORAGE_CLASS_EKS
                                                    echo "==================================================================="
                                                    echo "$PRODUCT_NAME deployed successfully on $PLATFORM_NAME platform !!!!"
                                                    echo "==================================================================="                                 
                                                '''
                                            }
                                        }
                                    }
                                }catch(error){
                                    throw error
                                }
                            }else {
                                try {
                                    withCredentials([string(credentialsId: 'XLD_LIC', variable: 'XLD_LICENSE')]) {
                                        withCredentials([string(credentialsId: 'XLD_KEYSTORE', variable: 'XLD_KEYSTORE')]) {
                                            withCredentials([string(credentialsId: 'XLD_PASS_PHRASE', variable: 'XLD_PASS_PHRASE')]) {
                                                sh '''
                                                    echo "Installing $PRODUCT_NAME on $PLATFORM_NAME platform"
                                                    /usr/local/bin/oc login --username=kubeadmin --password=$OPENSHIFT_AWS_CRED --server=$OPENSHIFT_AWS_SERVER_URL --insecure-skip-tls-verify
                                                    /usr/local/bin/oc project pipeline
                                                    /usr/local/bin/helm install --generate-name *.tgz --set route.hosts[0]=$HOST_NAME_XLD_OPENSHIFT --set xldLicense=${XLD_LICENSE} --set RepositoryKeystore=${XLD_KEYSTORE} --set KeystorePassphrase=${XLD_PASS_PHRASE} --set Persistence.StorageClass=$STORAGE_CLASS_EKS 
                                                    echo "==================================================================="
                                                    echo "$PRODUCT_NAME deployed successfully on $PLATFORM_NAME platform !!!!"
                                                    echo "==================================================================="                            
                                                '''
                                            }
                                        }
                                    }
                                }catch(error){
                                    throw error
                                }
                            }
                        }
                    }                    
                }
                stage('Openshift') {
                    when {
                        anyOf {
                            expression { params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Release' && params.PLATFORM == 'Openshift_Onprem' }
                            expression { params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Deploy' && params.PLATFORM == 'Openshift_Onprem' }                      
                        }
                    }
                    steps {
                        script {
                            if ( env.PRODUCT_NAME == 'XL Release' && env.PLATFORM_NAME == 'Openshift_Onprem') {
                                try {
                                    withCredentials([string(credentialsId: 'XLR_LIC', variable: 'XLR_LICENSE')]) {
                                        withCredentials([string(credentialsId: 'XLR_KEYSTORE', variable: 'XLR_KEYSTORE')]) {
                                            withCredentials([string(credentialsId: 'XLR_PASS_PHRASE', variable: 'XLR_PASS_PHRASE')]) {
                                                sh '''
                                                    echo "Installing $PRODUCT_NAME on $PLATFORM_NAME platform"
                                                    /usr/local/bin/oc login --username=kubeadmin --password=$OPENSHIFT_AWS_CRED --server=$OPENSHIFT_AWS_SERVER_URL
                                                    /usr/local/bin/oc project pipeline
                                                    /usr/local/bin/helm install --generate-name *.tgz --set route.hosts[0]=$HOST_NAME_XLR_OPENSHIFT --set xlrLicense=${XLR_LICENSE} --set RepositoryKeystore=${XLR_KEYSTORE} --set KeystorePassphrase=${XLR_PASS_PHRASE} --set Persistence.StorageClass=$STORAGE_CLASS_EKS
                                                    echo "==================================================================="
                                                    echo "$PRODUCT_NAME deployed successfully on $PLATFORM_NAME platform !!!!"
                                                    echo "==================================================================="                                 
                                                '''
                                            }
                                        }
                                    }
                                }catch(error){
                                    throw error
                                }
                            }else {
                                try {
                                    withCredentials([string(credentialsId: 'XLD_LIC', variable: 'XLD_LICENSE')]) {
                                        withCredentials([string(credentialsId: 'XLD_KEYSTORE', variable: 'XLD_KEYSTORE')]) {
                                            withCredentials([string(credentialsId: 'XLD_PASS_PHRASE', variable: 'XLD_PASS_PHRASE')]) {
                                                sh '''
                                                    echo "Installing $PRODUCT_NAME on $PLATFORM_NAME platform"
                                                    /usr/local/bin/oc login --username=kubeadmin --password=$OPENSHIFT_AWS_CRED --server=$OPENSHIFT_AWS_SERVER_URL
                                                    /usr/local/bin/oc project pipeline
                                                    /usr/local/bin/helm install --generate-name *.tgz --set route.hosts[0]=$HOST_NAME_XLD_OPENSHIFT --set xldLicense=${XLD_LICENSE} --set RepositoryKeystore=${XLD_KEYSTORE} --set KeystorePassphrase=${XLD_PASS_PHRASE} --set Persistence.StorageClass=$STORAGE_CLASS_EKS 
                                                    echo "==================================================================="
                                                    echo "$PRODUCT_NAME deployed successfully on $PLATFORM_NAME platform !!!!"
                                                    echo "==================================================================="                            
                                                '''
                                            }
                                        }
                                    }
                                }catch(error){
                                    throw error
                                }
                            }
                        }
                    }                    
                }
                stage('EKS') {
                    when {
                        anyOf {
                            expression { params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Release' && params.PLATFORM == 'EKS' }
                            expression { params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Deploy' && params.PLATFORM == 'EKS' }                    
                        }
                    }
                    steps {
                        script {
                            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS_CRED', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                                withKubeConfig(caCertificate: '', clusterName: 'devops-pipeline', contextName: 'spaliwal@devops-pipeline.eu-west-2.eksctl.io', credentialsId: 'EKS_CONFIG', namespace: 'pipeline', serverUrl: 'https://048C1D825717485A7D739C95C0184F18.gr7.eu-west-2.eks.amazonaws.com') {
                                    if ( env.PRODUCT_NAME == 'XL Release' && env.PLATFORM_NAME == 'EKS' ) {
                                        try {
                                            echo "Installing $PRODUCT_NAME chart on $PLATFORM_NAME platform"
                                            withCredentials([string(credentialsId: 'XLR_LIC', variable: 'XLR_LICENSE')]) {
                                                withCredentials([string(credentialsId: 'XLR_KEYSTORE', variable: 'XLR_KEYSTORE')]) {
                                                    withCredentials([string(credentialsId: 'XLR_PASS_PHRASE', variable: 'XLR_PASS_PHRASE')]) {
                                                    sh '''
                                                            /usr/local/bin/kubectl config set-context --current --namespace=pipeline
                                                            /usr/local/bin/helm install --generate-name *.tgz --set ingress.hosts[0]=$HOST_NAME_XLR_EKS --set haproxy-ingress.controller.service.type=${LOADBALANCER} --set xlrLicense=${XLR_LICENSE} --set RepositoryKeystore=${XLR_KEYSTORE} --set KeystorePassphrase=${XLR_PASS_PHRASE} --set Persistence.StorageClass=$STORAGE_CLASS_EKS
                                                            sleep 5
                                                            /usr/local/bin/kubectl get svc
                                                            echo "==================================================================="
                                                            echo "$PRODUCT_NAME deployed successfully on $PLATFORM_NAME platform !!!!"
                                                            echo "==================================================================="
                                                    '''
                                                    } 
                                                }
                                            }   
                                        }catch(error) {
                                            throw error
                                        }
                                    } else {
                                        try {
                                            echo "Installing $PRODUCT_NAME chart on $PLATFORM_NAME platform"
                                            withCredentials([string(credentialsId: 'XLD_LIC', variable: 'XLD_LICENSE')]) {
                                                withCredentials([string(credentialsId: 'XLD_KEYSTORE', variable: 'XLD_KEYSTORE')]) {
                                                    withCredentials([string(credentialsId: 'XLD_PASS_PHRASE', variable: 'XLD_PASS_PHRASE')]) {
                                                        sh '''
                                                            /usr/local/bin/kubectl config set-context --current --namespace=pipeline
                                                            /usr/local/bin/helm install --generate-name *.tgz --set ingress.hosts[0]=$HOST_NAME_XLD_EKS --set haproxy-ingress.controller.service.type=${LOADBALANCER} --set xldLicense=${XLD_LICENSE} --set RepositoryKeystore=${XLD_KEYSTORE} --set KeystorePassphrase=${XLD_PASS_PHRASE} --set Persistence.StorageClass=$STORAGE_CLASS_EKS
                                                            sleep 5
                                                            /usr/local/bin/kubectl get svc
                                                            echo "==================================================================="
                                                            echo "$PRODUCT_NAME deployed successfully on $PLATFORM_NAME platform !!!!"
                                                            echo "==================================================================="
                                                        '''
                                                    }
                                                }
                                            }
                                        }catch(error){
                                            throw error
                                        }
                                    }
                                }    
                            }
                        }
                    }                
                }
                stage('Onprem') {
                    when {
                        anyOf {
                            expression { params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Release' && params.PLATFORM == 'Onprem' }
                            expression { params.INSTALL_CHART == 'YES' && params.PRODUCT == 'XL Deploy' && params.PLATFORM == 'Onprem' }
                        }
                    }
                    steps {
                        script {
                            withCredentials([usernamePassword(credentialsId: 'ON_PREM_K8S', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: 'kubernetes-admin@kubernetes', credentialsId: 'ON_PREM_K8S', namespace: 'default', serverUrl: 'https://172.16.16.21:6443') {
                                    if ( env.PRODUCT_NAME == 'XL Release' && env.PLATFORM_NAME == 'Onprem' ) {                                
                                        try {                                 
                                            echo "Installing $PRODUCT_NAME chart on $PLATFORM_NAME platform"
                                            withCredentials([string(credentialsId: 'XLR_LIC', variable: 'XLR_LICENSE')]) {
                                                withCredentials([string(credentialsId: 'XLR_KEYSTORE', variable: 'XLR_KEYSTORE')]) {
                                                    withCredentials([string(credentialsId: 'XLR_PASS_PHRASE', variable: 'XLR_PASS_PHRASE')]) {
                                                        sh '''
                                                            /usr/local/bin/helm install --generate-name *.tgz --set ingress.hosts[0]=$HOST_NAME_XLR_ONPREM --set xlrLicense=${XLR_LICENSE} --set RepositoryKeystore=${XLR_KEYSTORE} --set KeystorePassphrase=${XLR_PASS_PHRASE} --set Persistence.StorageClass=$STORAGE_CLASS_ONPREM
                                                            sleep 5
                                                            /usr/local/bin/kubectl get svc
                                                            echo "==================================================================="
                                                            echo "$PRODUCT_NAME deployed successfully on $PLATFORM_NAME platform !!!!"
                                                            echo "==================================================================="
                                                        '''
                                                    }
                                                } 
                                            }
                                        }catch(error) {
                                            throw error
                                        }
                                    }else {
                                        try {
                                            echo "Installing $PRODUCT_NAME chart on $PLATFORM_NAME platform"
                                            withCredentials([string(credentialsId: 'XLD_LIC', variable: 'XLD_LICENSE')]) {
                                                withCredentials([string(credentialsId: 'XLD_KEYSTORE', variable: 'XLD_KEYSTORE')]) {
                                                    withCredentials([string(credentialsId: 'XLD_PASS_PHRASE', variable: 'XLD_PASS_PHRASE')]) {
                                                        sh '''
                                                            /usr/local/bin/helm install --generate-name *.tgz --set ingress.hosts[0]=$HOST_NAME_XLD_ONPREM --set xlrLicense=${XLD_LICENSE} --set RepositoryKeystore=${XLD_KEYSTORE} --set KeystorePassphrase=${XLD_PASS_PHRASE} --set Persistence.StorageClass=$STORAGE_CLASS_ONPREM    
                                                            sleep 5
                                                            /usr/local/bin/kubectl get svc
                                                            echo "==================================================================="
                                                            echo "$PRODUCT_NAME deployed successfully on $PLATFORM_NAME platform !!!!"
                                                            echo "==================================================================="
                                                        '''
                                                    }
                                                }
                                            }
                                        }catch(error) {
                                            throw error
                                        }
                                    } 
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
        // success {
        //     mail bcc: '', body: "<b>Example</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "SUCCESS CI: Project name -> ${env.JOB_NAME}", to: "chandra606@gmail.com";
        // }
        // failure {
        //     mail bcc: '', body: "<b>Example</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "ERROR CI: Project name -> ${env.JOB_NAME}", to: "chandra606@gmail.com";
        // }
    }
}
