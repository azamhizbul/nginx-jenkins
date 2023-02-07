import org.jenkinsci.plugins.pipeline.modeldefinition.Utils 

//GIT CHECKOUT
def git_credentials_id = scm.userRemoteConfigs[0].credentialsId
def git_repo = scm.userRemoteConfigs[0].url
def git_branch = scm.branches[0].name
def gitCommitId
def resultlog

//PREPARE 
def env_file = 'local-env'
def nexus_credentials_id = 'dockerhub_azam'
def nexus_base_url = 'https://nexus.posindonesia.co.id'
def appFullVersion

def nexus_docker_url_ip = "10.24.7.10:8087"
def nexus_docker_url = "docker.posindonesia.co.id"
def container_repo = "azamhizbul"

def pomappName = "nginx-jenkins"

// Docker host Deployment
def ssh_credentials_id_prod = "	server-prod-validasiposaja"
def ssh_host_prod = "10.29.41.56"
def ssh_credentials_id = "dev-kurlog-onprem"
def ssh_host = "10.60.64.47"
def remote = [:]

def isDevelopment = false
def isProduction = false
def skipBuildDocker = false
def skipKubernetDev = false
def skipKubernetProd = false

node {
    stage('Checkout') {
        cleanWs()
        git url: "${git_repo}", branch: "${git_branch}", credentialsId: "${git_credentials_id}"
        resultlog = sh(returnStdout: true, script: 'git log  -1 --pretty=%B')
        // resultlog = resultlog.toLowerCase()
        // env.M2_HOME = "/opt/maven"
        // env.PATH="/opt/oc:${env.M2_HOME}/bin:${env.PATH}"
    }

    stage('Prepare'){
        echo resultlog
        if(resultlog.contains("deployDev")) {
            isDevelopment = true
        }
        if(resultlog.contains("deployProd")) {
            isProduction = true
        }
        if(resultlog.contains("skipBuild")){
            skipBuildDocker = true
        } 
        if(resultlog.contains("kubernetDev")){
            skipKubernetDev = true
        } 
        if(isProduction) {
            isDevelopment = false
        }
        sh "echo development: ${isDevelopment}, production: ${isProduction}, skipBuilDockerDev: ${skipBuildDocker}"
    }   

    stage('Environment') {

        gitCommitId = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
        echo "pomappName: '${pomappName}', appFullVersion:'${appFullVersion}', gitCommitId:'${gitCommitId}'"
    
    }

 
    stage('Build Docker'){
        if(skipBuildDocker)
        {
            echo 'SKIP Docker image Dev'
            // Utils.markStageSkippedForConditional('Docker image DEV')
        }
        else
        {
        
        sh """
            docker build -t ${pomappName} .
            """
        sh """
            docker tag "${pomappName}" "${container_repo}/${pomappName}:latest"
           """
        
        }
    }
    
    stage('Push image'){
        if(skipBuildDocker)
        {
            echo 'SKIP Push image Dev'
            // Utils.markStageSkippedForConditional('Push image DEV')
        }
        else  
        {
        withEnv(["CREDENTIALID=${nexus_credentials_id}"]) {
           withCredentials([[$class: 'UsernamePasswordMultiBinding',
                credentialsId: "${CREDENTIALID}",
                usernameVariable: 'nexus_username', passwordVariable: 'nexus_password']]) { 
                sh """
                    docker login --username=${nexus_username} --password='${nexus_password}' 
                    docker push ${container_repo}/${pomappName}:latest
                    docker logout
                """

                sh """
                     set -e
                     set -x
                     docker rmi ${pomappName}
                     docker rmi ${container_repo}/${pomappName}:latest
                     
                """
                }
        }
        }
    }

    stage ('Deploy Docker'){
        def checkContainer = false
        try {
            sh "docker ps -a -f name=${pomappName}"
            checkContainer = true
        }catch (e) {
            checkContainer = false
            echo 'No restart deployment' + e.toString()
        }

        echo "${checkContainer}"
        if(checkContainer){
            sh """ docker stop ${pomappName} """
            sh """ docker rm ${pomappName} """
        }

        sh """ docker run -d -p 3000:80 --name=${pomappName} ${container_repo}/${pomappName}:latest """
    }

}


def renameInFile(variable, newvalue, filename)
{ 
    content = readFile(filename)
    content = content.replaceAll(variable, newvalue)
    writeFile file: filename, text: content
}
