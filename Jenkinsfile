node {
    def project = 'howard-west-1'
    def websiteName = 'searchlitestarter'
    def branchName = "${env.BRANCH_NAME}".replace("/", "_")
    def feImageTag = "us.gcr.io/${project}/${websiteName}-frontend:${branchName}.${env.BUILD_NUMBER}"
    def feBuildDir = "angular-client"
    def beImageTag = "us.gcr.io/${project}/${websiteName}-backend:${branchName}.${env.BUILD_NUMBER}"
    def beBuildDir = "backend"
    checkout scm

    stage 'Build image'
    sh("docker build -t ${feImageTag} ${feBuildDir}")
    sh("docker build -t ${beImageTag} ${beBuildDir}")

    stage 'Push image to registry'
    sh("gcloud docker -- push ${feImageTag}")
    sh("gcloud docker -- push ${beImageTag}")

    stage "Deploy Application"
    switch (env.BRANCH_NAME) {
        // Roll out to production
        case "master":
        // Change deployed image to the one we just built
        sh("sed -i.bak 's#us.gcr.io/howard-west-search/searchlite-backend:latest#${beImageTag}#' ./k8s/production/*.yaml")
        sh("sed -i.bak 's#us.gcr.io/howard-west-search/searchlite-frontend:latest#${feImageTag}#' ./k8s/production/*.yaml")
        sh("kubectl --namespace=${websiteName} apply -f k8s/services/")
        sh("kubectl --namespace=${websiteName} apply -f k8s/production/")
        sh("kubectl --namespace=${websiteName} get service/frontend")
        break

        default:
        echo "Branch built images ${feImageTag} and ${beImageTag} from branch ${env.BRANCH_NAME} but they were not deployed.  Use gcloud docker -- pull ${feImageTag} to get image on your local host."
    }
}
