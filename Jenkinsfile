podTemplate(label: 'nsqr-analytics',
    serviceAccount: 'jenkins-agent',
    containers: [
        containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:latest', command: 'cat', ttyEnabled: true)
    ], 
    envVars:[
        envVar(key: 'REGISTRY', value: 'docker.io/nsqr')
    ],
    volumes: [
        hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
        hostPathVolume(mountPath: '/root/.docker', hostPath: '/home/jenkins/.docker') 
    ]) {
    
    node('nsqr-analytics') {

        stage('Checkout') {
            checkout scm
            sh 'whoami'
        }
        stage('Docker login') {
            container('docker'){
                withCredentials([usernamePassword(credentialsId: 'hmitchellnsqrio-dockerhub', passwordVariable: 'REGISTRY_PASS', usernameVariable: 'REGISTRY_USER')]){
                    sh 'echo $REGISTRY_PASS | docker login $REGISTRY -u $REGISTRY_USER --password-stdin'
                }
            }
        }

        stage('Build') { 
            container('func'){
                def src_root = "${env.WORKSPACE}/mortgages"
                dir(src_root) {
                    sh "func build --verbose"
                }
            }
        }

        stage('Test') {
            container('func'){
                withCredentials([usernamePassword(credentialsId: 'hmitchellnsqrio-dockerhub', usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PASS')]) {

                    def src_root = "${env.WORKSPACE}/mortgages"
                    dir(src_root) {
                            sh 'func deploy --verbose --registry $REGISTRY'
                    }
                }
            }
        }
    }
}