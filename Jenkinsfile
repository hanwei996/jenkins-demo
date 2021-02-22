def label = "slave-pipeline"
podTemplate(label: label, cloud: 'kubernetes') {
    node(label) {
        // 定义第一个stage， 完成克隆源码的任务
        stage('checkout git') {
            checkout([$class: 'GitSCM', branches: [[name: '*/serverless']], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/hanwei996/jenkins-demo.git']]])
        }

        // 添加第二个stage， 运行源码打包命令
        stage('Package') {
                container("maven") {
                    sh "mvn package -B -DskipTests"
                }
        }

        // 添加第三个stage, 运行容器镜像构建和推送命令， 用到了environment中定义的groovy环境变量
        stage('Image Build And Publish') {
                container("kaniko") {
                    sh "kaniko -f `pwd`/Dockerfile -c `pwd` --destination=registry.cn-hangzhou.aliyuncs.com/ad_test/jenkins-demo:serverless"
                }
        }
//
//        stage('Deploy to Kubernetes') {
//            parallel {
//                stage('Deploy to Production Environment') {
//                    when {
//                        expression {
//                            "$BRANCH" == "serverless"
//                        }
//                    }
//                    steps {
//                        container('kubectl') {
//                            step([$class: 'KubernetesDeploy', apiServerUrl: 'https://kubernetes.default:6443', credentialsId: '7774c063-347d-4ab0-98a6-7318fe6df8e8', config: 'deployment.yaml', variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG'])
//                        }
//                    }
//                }
//                stage('Deploy to Staging001 Environment') {
//                    when {
//                        expression {
//                            "$BRANCH" == "latest"
//                        }
//                    }
//                    steps {
//                        container('kubectl') {
//                            step([$class: 'KubernetesDeploy', apiServerUrl: 'https://kubernetes.default:6443', credentialsId: '7774c063-347d-4ab0-98a6-7318fe6df8e8', config: 'deployment.yaml', variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG'])
//                        }
//                    }
//                }
//            }
//        }
    }
}
