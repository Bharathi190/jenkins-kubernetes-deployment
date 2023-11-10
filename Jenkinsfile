def FAILED_STAGE
pipeline {
    
        options {

    buildDiscarder(
        logRotator(
            // number of build logs to keep
            numToKeepStr:'3',
            // history to keep in days
            daysToKeepStr: '3',
            // artifacts are kept for days
            artifactDaysToKeepStr: '3',
            // number of builds have their artifacts kept
            artifactNumToKeepStr: '3'
        )
    )
}

    agent any

    environment {
        DOCKER_IMAGE_TAG = "0.1"
        KUBECONFIG = credentials('kubeconfig')
        recipientEmails = "mohana.a@intainft.com"
        //, bharathi.m@intainft.com, rajasekar.sv@intainft.com, sanjay.nishank@intainft.com, srihari.sardena@intainft.com"        
    }    

    stages {
    	stage('Git Checkout') {
        	steps {
        	                script {
                    FAILED_STAGE=env.STAGE_NAME
                    echo "stage 1"
                }
            	git branch: 'main', credentialsId: 'git-pat', url: 'https://github.com/mohana-intain/intain-va-node-app.git'

        	}
    	}
        stage('Docker Build') {
            steps {
            script {
                    FAILED_STAGE=env.STAGE_NAME
                    echo "stage 2"
                }
                sh 'docker build -t intainregistry/intainva-external-app:${DOCKER_IMAGE_TAG}.${BUILD_NUMBER} .'
                sh 'docker build -t intainregistry/intainva-external-app:latest .'

            }
        }
        stage('Docker Push') {
            steps {
            script {
                    FAILED_STAGE=env.STAGE_NAME
                    echo "stage 3"
                }
                withCredentials([usernamePassword(credentialsId: 'docker_pat', passwordVariable: 'docker_patPassword', usernameVariable: 'docker_patUser')]) {
                    sh "docker login -u ${env.docker_patUser} -p ${env.docker_patPassword}"
                    sh 'docker push intainregistry/intainva-external-app:${DOCKER_IMAGE_TAG}.${BUILD_NUMBER}'
                    sh 'docker push intainregistry/intainva-external-app:latest'

                }
                
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                FAILED_STAGE=env.STAGE_NAME
                    echo "stage 4"
                    // Get the Kubernetes configuration file path
                    def kubeConfigPath = sh(script: "echo \${KUBECONFIG}", returnStdout: true).trim()

                    // Use kubectl to apply your Kubernetes manifests
                    sh "sed -i \"s|{{image}}|intainregistry/intainva-external-app:${DOCKER_IMAGE_TAG}.${BUILD_NUMBER}|g\" deployment.yaml"
                    sh "cat deployment.yaml"
                    sh "kubectl --kubeconfig=${kubeConfigPath} apply -f deployment.yaml"
                    sh "kubectl --kubeconfig=${kubeConfigPath} apply -f service.yaml"
//                    sh 'kubectl set image deployments/intainva-external-app node-external-app=intainregistry/intainva-external-app:${DOCKER_IMAGE_TAG}.${BUILD_NUMBER}'

                }
               // catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                  //  sh "exit 1"
                //}
            }
        }
        stage('Delete images from local registry') {
            steps {
            script {
                FAILED_STAGE=env.STAGE_NAME
                    echo "stage 5"
                    }
                sh "docker rmi --force \$(docker images --format '{{.Repository}}:{{.Tag}}' | grep 'intainregistry/intainva-external-app:${DOCKER_IMAGE_TAG}.${BUILD_NUMBER}')"
                sh "docker rmi --force \$(docker images --format '{{.Repository}}:{{.Tag}}' | grep 'intainregistry/intainva-external-app:latest')"
            }
        }
    }
    post{
        success{
            mail to: "${recipientEmails}",
            subject: "Jenkins Build - Intain VA Node app is :${currentBuild.currentResult}",
            body: "Hi Team,\n\nDocker Image Tag Deployed Is: intainregistry/intainva-external-app:${DOCKER_IMAGE_TAG}.${BUILD_NUMBER}\n${currentBuild.currentResult}: Job ${env.JOB_NAME}\n\nMore Info can be found here: ${env.BUILD_URL}\n Display Name of Current Build: ${currentBuild.fullDisplayName}"          
        }
//${currentBuild.fullDisplayName}"
        failure{
            mail to: "${recipientEmails}",
            subject: "Jenkins Build - Intain VA Node app is :${currentBuild.currentResult}",
            body: "Hi Team,\n\nThis Docker Image Tag  intainregistry/intainva-external-app:${DOCKER_IMAGE_TAG}.${BUILD_NUMBER} is not deployed because of Failure\nFailed stage name: ${FAILED_STAGE}\n${currentBuild.currentResult}: Job ${env.JOB_NAME}\n\nMore Info can be found here: ${env.BUILD_URL}\n Display Name of Current Build: ${currentBuild.fullDisplayName}"
        }

    }
}
