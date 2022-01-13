pipeline {
    agent any
    environment {
        def config = readJSON file: 'package.json'
        VERSION = "${config.version}"
        PORT=3000
        NAME="${env.JOB_NAME}"
        BuildInfoUrl="http://10.200.0.128:80/project/jenkins/buildInfo"
        Build_Id="${env.BUILD_ID}"
        TimeStr = new Date().format('yyyyMMddHHmm')
    }
    stages {
        stage("编译打包") {
            steps {
                script {
                    def ProjectBranch = env.GIT_BRANCH.split("/")[1]
                    def packageManager = "/usr/local/bin/npm"
                    def START_SCRIPT= ""
                    sh "curl -i -X POST -H \"'Content-type':'application/json'\" -d \'{\"buildid\":\"${env.BUILD_ID}\",\"jenkinsjobname\":\"${env.JOB_NAME}\",\"buildurl\":\"${env.BUILD_URL}\",\"branchname\":\"${env.GIT_BRANCH}\",\"images\":\"\",\"status\":\"构建中\",\"steps\":\"start\"}\' ${BuildInfoUrl}"
                    if (ProjectBranch == "master"){

                        def START_SCRIPT_UAT= "release:uat"
                        sh "npm config set registry http://npm.wenwo.com && ${packageManager} install && ${packageManager}  run build:uat"

                        sh 'tar czf Data-uat-${VERSION}.tgz .nuxt config  static  node_modules  nuxt.config.js  package.json'
                        sh 'tar czf Static-Data-${VERSION}.tgz .nuxt'
                        sh " curl -F \"loadFile=@Static-Data-${VERSION}.tgz\" -F \"fileName=Static-Data-${VERSION}.tgz\" -F \"appName=${NAME}\" -F \"buildId=${Build_Id}\" -X POST \"http://10.200.0.128:80/upFile\" "

                        sh "echo '' > Dockerfile"
                        sh "echo 'FROM registry.cn-beijing.aliyuncs.com/awyl/nodejs:v1' >> Dockerfile"
                        sh "echo 'COPY Data-uat-${VERSION}.tgz Data-uat-${VERSION}.tgz' >> Dockerfile"
                        sh "echo 'RUN [\"mkdir\",\"-p\",\"/www/\"]' >> Dockerfile"
                        sh "echo 'RUN [\"tar\",\"-xf\",\"Data-uat-${VERSION}.tgz\",\"-C\",\"/www/\"]' >> Dockerfile"
                        sh "echo 'RUN [\"rm\",\"-f\",\"Data-uat-${VERSION}.tgz\"]' >> Dockerfile"
                        sh "echo 'WORKDIR \"/www/\"' >> Dockerfile"
                        sh "echo 'ENTRYPOINT [\"npm\",\"run\",\"${START_SCRIPT_UAT}\"]' >> Dockerfile"
                        sh "echo 'EXPOSE ${PORT}' >> Dockerfile"
                        sh 'cat Dockerfile'
                        sh "docker build -t h5/${NAME}:${VERSION}-aut-${TimeStr} ."
                        sh "ls -alh "
                        sh "ls -alh .nuxt/dist/client/css"
                        sh "rm -rf Data-uat-${VERSION}.tgz node_modules/.cache"
                        sh "ls -alh "
                        
                        START_SCRIPT = "release"
                        sh " ${packageManager}  run build"
                        sh "ls -alh .nuxt/dist/client/css"
                    }else if (ProjectBranch == "rebuild-test"){
                        START_SCRIPT = "release:rebuild-test"
                        sh "npm config set registry http://npm.wenwo.com && ${packageManager}  run install-server && ${packageManager}  run build:rebuild-test"
                    }else if (ProjectBranch == "test"){
                        START_SCRIPT = "release:test"
                        sh "npm config set registry http://npm.wenwo.com && ${packageManager} install && ${packageManager}  run build:test"
                    }else if (ProjectBranch == "rebuild-dev"){
                        START_SCRIPT = "release:rebuild-dev"
                        sh "npm config set registry http://npm.wenwo.com && ${packageManager} install && ${packageManager}  run build:rebuild-dev"
                    }else if (ProjectBranch == "uat"){

                    }else {
                        START_SCRIPT = "start"
                        sh "npm config set registry http://npm.wenwo.com && ${packageManager} install"
                    }
                    echo VERSION
                    sh 'tar czf Data-${VERSION}.tgz .'
                    //sh 'tar czf Static-Data-${VERSION}.tgz .nuxt'
                    //sh " curl -F \"loadFile=@Static-Data-${VERSION}.tgz\" -F \"fileName=Static-Data-${VERSION}.tgz\" -F \"appName=${NAME}\" -F \"buildId=${Build_Id}\" -X POST \"http://10.200.0.128:80/upFile\" "
                    def projectBranch = env.GIT_BRANCH.split("/")[1]
                    sh "echo '' > Dockerfile"
                    sh "echo 'FROM registry.cn-beijing.aliyuncs.com/awyl/nodejs:v1' >> Dockerfile"
                    sh "echo 'COPY Data-${VERSION}.tgz Data-${VERSION}.tgz' >> Dockerfile"
                    sh "echo 'RUN [\"mkdir\",\"-p\",\"/www/vendors\"]' >> Dockerfile"
                    sh "echo 'RUN [\"tar\",\"-xf\",\"Data-${VERSION}.tgz\",\"-C\",\"/www/vendors\"]' >> Dockerfile"
                    sh "echo 'RUN [\"rm\",\"-f\",\"Data-${VERSION}.tgz\"]' >> Dockerfile"
                    sh "echo 'RUN [\"cp\",\"/www/vendors/config-dev.json\",\"/www/config.json\"]' >> Dockerfile"
                    sh "echo 'WORKDIR \"/www/vendors\"' >> Dockerfile"
                    sh "echo 'ENTRYPOINT [\"npm\",\"run\",\"${START_SCRIPT}\"]' >> Dockerfile"
                    sh "echo 'EXPOSE ${PORT}' >> Dockerfile"
                    sh 'cat Dockerfile'
                    sh "docker build -t h5/${NAME}:${VERSION} ."
                }
            }
        }
        stage("上传镜像") {
            steps {
                script {
                    def projectBranch = env.GIT_BRANCH.split("/")[1]
                    def imageName="10.200.0.143:80/wenwo/${NAME}:${VERSION}-${projectBranch}-${env.BUILD_ID}-${TimeStr}"
                    def imageNameUat=""
                    if (projectBranch == "master"){
                        imageNameUat="10.200.0.143:80/wenwo/${NAME}:${VERSION}-uat-${env.BUILD_ID}-${TimeStr}"
                        sh "docker tag h5/${NAME}:${VERSION}-aut-${TimeStr} ${imageNameUat}"
                        sh "docker push ${imageNameUat}"
                        sh "docker rmi h5/${NAME}:${VERSION}-aut-${TimeStr} ${imageNameUat}"
                    }

                    sh "docker tag h5/${NAME}:${VERSION} ${imageName}"
                    sh "docker push ${imageName}"
                    sh "docker rmi h5/${NAME}:${VERSION} ${imageName}"
                    sh "curl -i -X POST -H \"'Content-type':'application/json'\" -d \'{\"buildid\":\"${env.BUILD_ID}\",\"jenkinsjobname\":\"${env.JOB_NAME}\",\"buildurl\":\"${env.BUILD_URL}\",\"branchname\":\"${env.GIT_BRANCH}\",\"images\":\"${imageName}\",\"uatImages\":\"${imageNameUat}\",\"staticData\":\"Static-Data-${VERSION}-${Build_Id}.tgz\",\"status\":\"构建成功\",\"steps\":\"end\"}\'  ${BuildInfoUrl}"
                }
            }
        }
        stage("清理空间") {
            steps {
                sh "ls -al"
                deleteDir()
                sh "ls -al"
            }
        }
    }
    post {
        failure {
            echo "failure"
            deleteDir()
            sh "curl -i -X POST -H \"'Content-type':'application/json'\" -d \'{\"buildid\":\"${env.BUILD_ID}\",\"jenkinsjobname\":\"${env.JOB_NAME}\",\"buildurl\":\"${env.BUILD_URL}\",\"branchname\":\"${env.GIT_BRANCH}\",\"images\":\"\",\"status\":\"构建失败\",\"steps\":\"end\"}\' ${BuildInfoUrl}"
        }
    }
}
