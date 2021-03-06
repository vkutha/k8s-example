node {
    checkout scm
    def DOCKER_HUB_ACCOUNT = 'bharathan1980'
    def DOCKER_IMAGE_NAME = 'k8s-example'
    
    echo 'Building Go App'
    stage("build") {
        docker.image("icrosby/jenkins-agent:kube").inside('-u root') {
            sh 'go build' 
        }
    }
    echo 'Testing Go App'
    stage("test") {
        docker.image('icrosby/jenkins-agent:kube').inside('-u root') {
            sh 'go test' 
            sh 'printenv'
        }
    }

    echo 'Building Docker image'
    
    stage('BuildImage') 
    def app = docker.build("${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${BUILD_TAG}", '.')

    echo 'Testing Docker image'
    stage("test image") {
        app.inside {
          sh '/test.sh'
        }
    }

    echo 'Pushing Docker Image'
    stage("Push")
    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
        app.push()
        app.push('${JENKINS_SERVER_COOKIE}')
    }

    echo "Deploying image"
    stage("Deploy") 
    docker.image('smesch/kubectl').inside{
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            //sh "kubectl --kubeconfig=$KUBECONFIG apply -f deployment.yaml --validate=false"
            sh 'kubectl --kubeconfig=$KUBECONFIG set image deployment/k8s-example k8s-example=bharathan1980/k8s-example:${BUILD_TAG}'
        }
    }
}
