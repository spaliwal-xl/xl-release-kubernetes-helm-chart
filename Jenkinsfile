def InstallChart (String release) {
    echo "installing ${release}"
    script {
        sh "helm install ${release} *.tgz --set ingress.hosts=${HostName} --set xlrLicense=${env.XLR_License} --set RepositoryKeystore=${env.KeyStore} --set KeystorePassphrase=${env.PassPhrase} --set Persistence.StorageClass=${StorageClass}"
    }
}

pipeline {

    agent none
    tools {
        helm 'helm-3.0.0' 
    }
    parameters {
        string(name: 'RELEASE_BRANCH_NAME', defaultValue: 'master', description: 'The branch from which to make the release')
        booleanParam(name: 'Install_Chart', defaultValue: false, description: 'Do you want to install the chart')
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '1'))
        timeout(time: 1, unit: 'HOURS')
        timestamps()
        retry(1)
    }

    environment {
        PARAMETERS_FILE = "${JENKINS_HOME}/parameters.groovy"
        NEXUS_USER = "admin"
        NEXUS_PASSWORD = "admin123"
        NEXUS_URL = "http://nexus-appserver.digitalai-testing.com:8081/repository/helm_repository/"
    }
    
    stages {
        stage('Checkout and Package') {
            agent any
            steps {
                script {
                    try {
                        sh "echo 'Release branch: ${params.RELEASE_BRANCH_NAME}'"
                        sh "git clone -b ${params.RELEASE_BRANCH_NAME} https://github.com/spaliwal-xl/xl-release-kubernetes-helm-chart"
                        sh "helm dependency update xl-release-kubernetes-helm-chart"
                        sh "helm package xl-release-kubernetes-helm-chart"
                    } catch (err) {
                        echo "Checkout Failed!"
                        throw err  
                    }
                }
            }
        }
        
        stage('Push to Nexus') {
            agent any
            steps {
                sh "echo Pushing the build"
                sh "curl -u ${NEXUS_USER}:${NEXUS_PASSWORD} ${NEXUS_URL} --upload-file *.tgz -v"
                load "${JENKINS_HOME}/parameters.groovy"
                script {
                    withKubeConfig([credentialsId: 'kubelove', serverUrl: 'https://172.16.16.36:6443']) {
                        if (params.Install_Chart) {
                            release = "dev"
                            InstallChart (release)
                        }
                    }
                }
            }
        }
        
        stage('Notify and Cleanup') {
            agent any
            steps {
                echo "Cleaning"
            }
        }
    }
}
