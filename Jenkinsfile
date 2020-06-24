pipeline {
    agent {
        kubernetes {
            label 'jenkins-test-k8s-pipeline'  // all your pods will be named with this prefix, followed by a unique id
            idleMinutes 1  // how long the pod will live after no jobs have run on it
            yamlFile 'build-pod.yaml'  // path to the pod definition relative to the root of our project 
        }
    }
    environment {
        DOCKER_REPO_CREDS = credentials('68527666-ec6d-4c9a-afb2-a9e7e9e78431')
        KUBECONFIG = credentials('kubeconfig')
        DOCKERTAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Build') {
            steps {  // no container directive is needed as the maven container is the default
                container('maven') {
                    sh "mvn clean package"   
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh "docker login -u ${env.DOCKER_REPO_CREDS_USR} -p ${env.DOCKER_REPO_CREDS_PSW}"  // Login   
                    sh "docker build -t jwenzel/jenkins-test-k8s-pipeline:dev ."  // when we run docker in this step, we're running it via a shell on the docker build-pod container,
                    sh "docker push jwenzel/jenkins-test-k8s-pipeline:${env.DOCKERTAG}"        // which is just connecting to the host docker deaemon
                }
            }
        }
        stage('Helm Deployment') {
            steps {
                container('helm') {
                    echo "Deploying to test"
                    sh "kubectl get pods --all-namespaces -o wide --kubeconfig ${env.KUBECONFIG}"
                    //sh("helm upgrade --install --force jenkins-test-k8s-pipeline --namespace test --set image.tag=${dockerTag} ./helm")
                }
            }
        }
    }
}


