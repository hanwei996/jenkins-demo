pipeline{
      // 定义groovy脚本中使用的环境变量
      environment{
        // 本示例中使用DEPLOY_TO_K8S变量来决定把应用部署到哪套容器集群环境中，如“Production Environment”， “Staging001 Environment”等
        IMAGE_TAG =  sh(returnStdout: true,script: 'echo $image_tag').trim()
        ORIGIN_REPO =  sh(returnStdout: true,script: 'echo $origin_repo').trim()
        REPO =  sh(returnStdout: true,script: 'echo $repo').trim()
        BRANCH =  sh(returnStdout: true,script: 'echo $branch').trim()
        API_SERVER_URL = sh(returnStdout: true,script: 'echo $api_server_url').trim()
      }

      // 定义本次构建使用哪个标签的构建环境，本示例中为 “slave-pipeline”
      agent{
        node{
          label 'slave-pipeline'
        }
      }

      // "stages"定义项目构建的多个模块，可以添加多个 “stage”， 可以多个 “stage” 串行或者并行执行
      stages{
	// 定义第一个stage， 完成克隆源码的任务
        stage('Git'){
          steps{
            git branch: '${BRANCH}', credentialsId: '', url: 'https://github.com/hanwei996/jenkins-demo.git'
          }
        }

	// 添加第二个stage， 运行源码打包命令
        stage('Package'){
          steps{
              container("maven") {
                  sh "mvn package -B -DskipTests"
              }
          }
        }

	// 添加第三个stage, 运行容器镜像构建和推送命令， 用到了environment中定义的groovy环境变量
        stage('Image Build And Publish'){
          steps{
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
				step([$class: 'KubernetesDeploy', apiServerUrl: '$API_SERVER_URL', credentialsId:'7774c063-347d-4ab0-98a6-7318fe6df8e8', config: 'deployment.yaml',variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG'])
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
				step([$class: 'KubernetesDeploy',  apiServerUrl: '$API_SERVER_URL', credentialsId:'7774c063-347d-4ab0-98a6-7318fe6df8e8', config: 'deployment.yaml',variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG'])
			}
                    }
                }
            }
        }
      }
    }
