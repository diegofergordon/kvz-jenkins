pipeline {
    agent any

    stages {
        stage('Download Files') {
            steps {
                configFileProvider([configFile(fileId: 'a44966b8-cda9-41a3-8cf2-3f9b5806088f', variable: 'CLIENTKEY'),
                                    configFile(fileId: 'dabe92e9-ccf9-422f-a903-7f20312dc7d7', variable: 'CLIENTCRT'),
                                    configFile(fileId: 'f6d45238-963d-4302-a944-a500d5622231', variable: 'CACRT')]) {
                    sh 'cp $CLIENTKEY $(pwd)@tmp/client.key'
                    sh 'cp $CLIENTCRT $(pwd)@tmp/client.crt'
                    sh 'cp $CACRT $(pwd)@tmp/ca.crt'
                } 
            }
        }
        stage('Download Sources') {
            steps {
                // Get some code from a GitHub repository
                git 'https://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@github.com/diegofergordon/kvz-api-bank.git'
            }
        }
        stage('Maven Build') {
            steps {
                configFileProvider([configFile(fileId: '91ad53be-557a-4851-9e24-5deafa780985', variable: 'MAVEN_SETTINGS')]) {
                    sh 'cp $MAVEN_SETTINGS $(pwd)/jfrog-settings.xml'
                    sh 'docker run --rm -i -v "$(pwd)":/home/kvz-api-bank-deploy -v /home/devops/maven-repository:/home/kvz-api-bank-deploy/repository -w /home/kvz-api-bank-deploy maven:3.6.3-openjdk-11 mvn -Dmaven.test.failure.ignore=true -s jfrog-settings.xml clean package'
                }  
            }
        }
        stage('Maven Deploy JFrog') {
            steps {
                configFileProvider([configFile(fileId: '91ad53be-557a-4851-9e24-5deafa780985', variable: 'MAVEN_SETTINGS')]) {
                    sh 'cp $MAVEN_SETTINGS $(pwd)/jfrog-settings.xml'
                    sh 'docker run --rm -i -v "$(pwd)":/home/kvz-api-bank-deploy -v /home/devops/maven-repository:/home/kvz-api-bank-deploy/repository -w /home/kvz-api-bank-deploy maven:3.6.3-openjdk-11 mvn -s jfrog-settings.xml deploy'
                }  
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t docker.internal.10.42.0.28.nip.io:5000/kvz-api-bank ."
            }
        }
        stage('Docker Push') {
            steps {
                sh "docker push docker.internal.10.42.0.28.nip.io:5000/kvz-api-bank"
            }
        }
        stage('Kubernetes Deploy') {
            steps {
                configFileProvider([configFile(fileId: 'aaffabc6-5b52-40f5-bfbd-9d4e15b70b56', variable: 'KUBECONFIG')]) {
                    sh 'minikube kubectl delete deployment kvz'
                    sh 'minikube kubectl -- apply -f deployment.yaml'
                } 
            }
        }
    }
}
