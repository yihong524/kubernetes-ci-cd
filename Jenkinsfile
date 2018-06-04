pipeline {
    agent {
        kubernetes {
            label: 'myPod'
            defaultContainer: 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
    labels:
        jenkins: kube-pod
spec:
    nodeSelector:
        kubernetes.io/hostname: ubuntu-node2
    containers:
    - name: docker
      image: docker:17.03
      imagePullPolicy: Always
      command:
      - cat
      tty: true
      volumeMounts:
      - name: docker
        mountPath: /var/run/docker.sock
    - name: kubectl
      image: 192.168.200.21:30797/kubectl:1.10.0
      imagePullPolicy: Always
      command:
      - cat
      tty: true
      volumeMounts:
      - name: kubectl
        mountPath: /root/.kube
    volumes:
    - name: docker
      hostPath:
        path: /var/run/docker.sock
    - name: kubectl
      secret:
        secretName: kube-config
"""
        }
    }
    stages {
        stage('checkout code') {
            checkout scm

            sh "git rev-parse --short HEAD > commit-id"

            tag = readFile('commit-id').replace("\n", "").replace("\r", "")
            appName = "hello-kenzan"
            registryHost = "192.168.200.21:30797/"
            imageName = "${registryHost}${appName}:${tag}"
            env.BUILDIMG=imageName
        }
        stage('build and Push image') {
            container('docker') {
                sh "docker build -t ${imageName} -f applications/hello-kenzan/Dockerfile applications/hello-kenzan"
                sh "docker push ${imageName}"
            }
        }
        stage('Deploy') {
            container('kubectl') {
                sh "sed 's#127.0.0.1:30400/hello-kenzan:latest#'$BUILDIMG'#' applications/hello-kenzan/k8s/deployment.yaml | kubectl apply -f -"
                sh "kubectl rollout status deployment/hello-kenzan"
            }
        }

    }
}