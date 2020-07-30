node {

    def project_id = "strategic-guru-280413"
    def appName = "events-sample-external"
    def tagName = "gcr.io/${project_id}/${appName}:${env.BUILD_NUMBER}"

    stage ('gitSCM') {
        git 'https://github.com/Pradeepv30/external-app.git'
    }

    stage ('Sonar-analysis') {
        script {
            def scannerhome = tool 'sonar-scanner'
            withSonarQubeEnv ("SonarQube") {
                sh "${scannerhome}/bin/sonar-scanner \
                    -Dsonar.projectKey=capstone \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://34.70.35.238:9000 \
                    -Dsonar.login=fff4c2b1eba697ece8018731b6e416e26ad0c043"
            }
        }
    }

    stage ('node-test') {
        sh 'npm install'
        sh 'npm test'
    }

    stage ('build-docker') {
        //sh "docker build -t gcr.io/${project_id}/${appName}:${BUILD_NUMBER} ."
        //sh "docker push gcr.io/${project_id}/${appName}:${env.BUILD_NUMBER}"
        sh "gcloud builds submit --tag=${tagName} ."
    }

    stage ('deploy-into-kubernetes') {
        sh "sed -i -e 's#image:.*#image: ${tagName}#g' externaldeployment.yaml"
        sh "gcloud container clusters get-credentials events-feed-cluster --zone us-central1-a --project strategic-guru-280413"
        //sh "kubectl create deployment ${appName} --image=gcr.io/${project_id}/${appName}:${env.BUILD_NUMBER}"
        sh "kubectl apply -f externaldeployment.yaml"
        sh "kubectl apply -f externalservice.yaml"
        sh 'kubectl get pods'
        //sh "kubectl expose deployment ${appName} --name=${appName}-service --type=LoadBalancer --port 80 --target-port 8080"
        sh "kubectl get service"
    }
}