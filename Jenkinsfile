pipeline {
    agent none
    options { skipDefaultCheckout() }
    environment {
        TERRAFORM_DIR = "."
    }
    stages {
        stage('checkout') {
            agent { label 'master' }
            steps {
                checkout scm
            }
        }
        stage('plan') {
            agent { label 'master' }
            steps {
                //dir("${TERRAFORM_DIR}") {
                    script {
                        wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
                            setupPythonVirtualEnv()
                            sh 'TF_IN_AUTOMATION=true terraform get -update=true'
                            sh 'TF_IN_AUTOMATION=true terraform init -input=false'
                            sh 'TF_IN_AUTOMATION=true terraform plan -out terraform.tfplan -input=false'
                        }
                    }
                //}
                //stash includes: "${TERRAFORM_DIR}/terraform.tfplan", name: "terraform-plan"
                stash includes: "terraform.tfplan", name: "terraform-plan"
            }
        }
        stage('approve-plan') {
            agent none
            steps {
                script {
                    milestone()
                    timeout(time: 5, unit: 'MINUTES') {
                        input "Have you reviewed terraform plan in jenkins console? Do you want to proceed with deployment?"
                    }
                }
            }
        }
        stage('apply') {
            agent { label 'master' }
            steps {
                //dir("${TERRAFORM_DIR}") {
                    unstash "terraform-plan"
                    script {
                        wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
                            sh "TF_LOG=DEBUG TF_LOG_PATH=terraform_apply_logs TF_IN_AUTOMATION=true terraform apply -input=false terraform.tfplan"
                            archiveArtifacts 'terraform_apply_logs'
                        }
                    }
                //}
            }
        }
    }
}

def setupPythonVirtualEnv() {
    // not used.
}