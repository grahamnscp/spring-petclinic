def dtr_creds = '083117b8-df15-4801-9cab-57e36b4c498a'
def dtr_url = 'dtr.joeg.dtcntr.net'
def qa_repo = 'admin/pet-clinic-test'
def prod_repo = 'admin/pet-clinic-prod'

def git_url = 'https://github.com/grdnrio/ee-jenkins.git'
def signing_key = '2cb7cdba-107e-4a3e-ba98-931b7066a121'

node("docker") {
    withCredentials([
        [$class: 'UsernamePasswordMultiBinding', credentialsId: dtr_creds, usernameVariable: 'DTR_USR', passwordVariable: 'DTR_PWD'],
        [$class: 'StringBinding', credentialsId: signing_key, variable: 'SIGNING_KEY'],
    ]){
        git url: "${git_url}"
          
        sh "git rev-parse HEAD > .git/commit-id"
        def commit_id = readFile('.git/commit-id').trim()
        println commit_id

        stage "Tagging"
        sh " \
        docker login -u ${DTR_USR} -p ${DTR_PWD} ${dtr_url} && \
        docker pull ${dtr_url}/${qa_repo}:latest && \
        docker tag ${dtr_url}/${qa_repo}:latest ${dtr_url}/${prod_repo}:${BUILD_NUMBER}"

        stage "publish"
        sh "export NOTARY_DELEGATION_PASSPHRASE=${SIGNING_KEY} && \
        notary -d ~/.docker/trust key import /home/jenkins/key.pem --role=targets/qa && \
        notary -d ~/.docker/trust key list"
        
        sh "export DOCKER_CONTENT_TRUST=1 \
        DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE=${SIGNING_KEY} && \
        docker login -u ${DTR_USR} -p ${DTR_PWD} ${dtr_url} && \
        docker push ${dtr_url}/${prod_repo}:${BUILD_NUMBER}"

        stage "qa-deploy"
        sh "cd /home/jenkins && \
        source env.sh && \
        docker service update --image ${dtr_url}/${prod_repo}:${BUILD_NUMBER} pets-test_petclinic"
    }
}