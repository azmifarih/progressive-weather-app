node('jnlp-slave') {
    env.NODEJS_HOME = "${tool 'Node'}"
    env.PATH="${env.NODEJS_HOME}/bin:${env.PATH}"

    stage('Prepare') {
        echo "1.Prepare Stage"
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/azmifarih/progressive-weather-app.git']]])
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }
    stage('Compile') {
        echo "2.Compile VueJs"
        sh "npm install"
        sh "npm run build"
    }
    stage('Build') {
        container('docker'){
            echo "3.Build Docker Image Stage"
            sh "docker build -t azmifarih/weatherapp:${build_tag} ."
        }
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword}"
            sh "docker push azmifarih/weatherapp:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "5. Deploy To K8S Cluster Stage"
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' weatherapp-kubernetes.yaml"
        sh "kubectl apply -f weatherapp-kubernetes.yaml --record"
    }
}
