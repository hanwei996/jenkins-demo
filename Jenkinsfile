pipeline {
    // 定义groovy脚本中使用的环境变量
    environment {
        // 本示例中使用DEPLOY_TO_K8S变量来决定把应用部署到哪套容器集群环境中，如“Production Environment”， “Staging001 Environment”等
        IMAGE_TAG = sh(returnStdout: true, script: 'echo $image_tag').trim()
        ORIGIN_REPO = sh(returnStdout: true, script: 'echo $origin_repo').trim()
        REPO = sh(returnStdout: true, script: 'echo $repo').trim()
        BRANCH = sh(returnStdout: true, script: 'echo $branch').trim()
        API_SERVER_URL = sh(returnStdout: true, script: 'echo $api_server_url').trim()
    }

    // 定义本次构建使用哪个标签的构建环境，本示例中为 “slave-pipeline”
    agent {
        kubernetes { // 连接k8s，并利用yamlFile创建jenkins slave
            cloud 'kubernetes' //cloud关联
            label 'slave-pipeline' //label一定要写
            // defaultContainer 'python36' //  stages和post步骤中默认用到的container。如需指定其他container，可用语法 container("java8"){...}
            idleMinutes 10 //所创建的pod在job结束后直到销毁前的等待时间
            // yamlFile "jenkins/jenkins_pod_template.yaml" // 指定创建pod时的yaml配置文件
        }
//            node{
//              label 'slave-pipeline'
//            }
    }

    // "stages"定义项目构建的多个模块，可以添加多个 “stage”， 可以多个 “stage” 串行或者并行执行
    stages {
        // 定义第一个stage， 完成克隆源码的任务
        stage('Git') {
            steps {
                git branch: 'serverless', credentialsId: '', url: 'https://github.com/AliyunContainerService/jenkins-demo.git'
            }
        }

        // 添加第二个stage， 运行源码打包命令
        stage('Package') {
            steps {
                container("maven") {
                    sh "mvn package -B -DskipTests"
                }
            }
        }

        // 添加第三个stage, 运行容器镜像构建和推送命令， 用到了environment中定义的groovy环境变量
        stage('Image Build And Publish') {
            steps {
                container("kaniko") {
                    sh "kaniko -f `pwd`/Dockerfile -c `pwd` --destination=${ORIGIN_REPO}/${REPO}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            parallel {
                stage('Deploy to Production Environment') {
                    when {
                        expression {
                            "$BRANCH" == "serverless"
                        }
                    }
                    steps {
                        container('kubectl') {
                            step([$class: 'KubernetesDeploy', authMethod: 'certs', apiServerUrl: '$API_SERVER_URL', credentialsId: 'k8sCertAuth', config: 'deployment.yaml', variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG'])
                        }
                    }
                }
                stage('Deploy to Staging001 Environment') {
                    when {
                        expression {
                            "$BRANCH" == "latest"
                        }
                    }
                    steps {
                        container('kubectl') {
                            step([$class: 'KubernetesDeploy', authMethod: 'certs', apiServerUrl: '$API_SERVER_URL', credentialsId: 'k8sCertAuth', config: 'deployment.yaml', variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG'])
                        }
                    }
                }
            }
        }
    }
}
