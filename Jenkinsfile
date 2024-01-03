def version

pipeline {
    agent any

    parameters {
        string(name: 'version', defaultValue: '1.2', description: 'Version parameter')
    }

    environment {
        CONTAINER_NAME = "cowsay-test"
        IMAGE_NAME = "cowsay"
        NETWORK_NAME = "cowsay"
        EMAIL_TO = "ganshmuelgreen@gmail.com"
        DOCKER_REGISTRY = "644435390668.dkr.ecr.eu-west-3.amazonaws.com/michal"
        EC2_HOST = "10.0.28.159"
    }

    post {
        always {
            cleanWc()
        }
        success {
            emailext body: "The build was success - #$BUILD_NUMBER", recipientProviders: [culprits(), developers()], subject: 'The build succeeded'
        }
        failure {
            emailext body: "The build was failed- #$BUILD_NUMBER", recipientProviders: [culprits(), developers()], subject: 'The build was a failure'
        }
    }

    stages {
        stage('Git'){
      steps{
        sshagent(['gitlab-ssh']) {
          checkout scm
        }
      }
    }
        stage('Existing of the branch') {
        steps {
        sshagent(['gitlab-ssh']){ 
            script {
        //    sh 'git config --get remote.origin.url'
            def branchName = "release/${params.version}"
            def exists = sh( script: "git ls-remote  --heads origin \"$branchName\" | grep -q \"$branchName\"", returnStatus: true)
            println exists
            if (exists == 0) {
                echo "Branch ${branchName} already exists"
                sh 'cd ~/workspace/versioned_cowsay && cat v.txt'
    //            sh "git fetch origin"
                    sh "git checkout ${branchName}"
    //            sh "git pull origin ${branchName}"
                    sh 'cd ~/workspace/versioned_cowsay && cat v.txt'
                } else {           
                //create a new branch
                echo "------------------create a new branch------------------"
                sh "git checkout -b ${branchName} origin/main"
                // update v.txt with new version number
                echo "----------------update v.txt with new version number------------------"
                sh "echo '\n${params.version}' >> v.txt"
                sh "cat v.txt"
                sh "git status"
                //commit and push changes to gitlab
                sh "git add v.txt"
                sh "git commit -m 'update version number for release branch' "
                sh "git push -u origin ${branchName}"
                echo "--------------commit and push changes to gitlab----------------"
                } 
            }
        }
        }
    }

        
        stage('Calculate last version') {
        when { expression {params.version != '' }}
        steps{
        sshagent(['gitlab-ssh']){
            script{
            version = sh(returnStdout: true, script: "sed -n '2p' v.txt").trim()

            def tag = "${version}."
            def latestTag = sh( returnStdout: true,
            script: "git tag --sort=-v:refname --list ${tag}* | head -n1 | sed 's/${tag}//' ",
            ).trim()
            if (!latestTag) {
                    echo "Didn't find matching  ${tag}* tag"
                    def line = sh(returnStdout: true, script: "sed -n '2p' v.txt " ).trim()
                    def updateTag = "${line}.0"
                    sh "sed -i '2s/.*/${updateTag}/' v.txt"
            } else {
                    def nextTag = latestTag.toInteger() + 1
                    def newTag = "${tag}${nextTag}"
                    echo "latest tag matching ${tag}* is ${tag}${latestTag}"
                    def line = sh(returnStdout: true, script: "sed -n '2p' v.txt ").trim()
                // def updateTag = "${line}.${newTag}"
                    sh "sed -i '2s/.*/${newTag}/' v.txt"  //updateTag
            }
                
            version = sh(returnStdout: true, script: "sed -n '2p' v.txt").trim()
            }
        }
        }
        }
        

        stage('Build') {
        when{
        expression { env.GIT_BRANCH != "origin/main" || params.version != "" }
        }
        steps {
            echo "Building stage"
            sh "docker build -t ${env.IMAGE_NAME}:${version}  ."     //updateTag in docker  and git 
            echo "Finishing building stage"
        }
        }

        stage('Run'){
        when{
        expression { env.GIT_BRANCH != "origin/main" || params.version != "" }
        }
        steps{
            echo "Running the container"
            sh "docker run -d --name ${env.CONTAINER_NAME} --rm --network=${env.NETWORK_NAME} ${env.IMAGE_NAME}:${version} "
            echo "Finishing running stage "
        }
        }
        
        stage('Test') {
            when{
            expression { env.GIT_BRANCH != "origin/main" || params.version != "" }
            }
            steps {
            echo 'start unit test stage'
            sh "sleep 5"
            sh "curl -I http://${env.CONTAINER_NAME}:8080"
            sh "docker stop ${env.CONTAINER_NAME}"
            echo 'Finish unit test stage'
            }
        }

        stage('Release') {
            steps {
                // Clean/reset any changes made to files, then `docker tag` and `docker push`.
                sh "docker tag ${env.IMAGE_NAME}:${version} ${env.DOCKER_REGISTRY}:${env.IMAGE_NAME}-${version}"
                sshagent(['ssh-to-ec2']) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${env.EC2_HOST} \"aws ecr get-login-password --region eu-west-3 | docker login --username AWS --password-stdin ${env.DOCKER_REGISTRY}\""
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${env.EC2_HOST} 'docker push ${env.DOCKER_REGISTRY}:${env.IMAGE_NAME}-${version}'"
                }
            }
        }

        stage('Git push tag') {
            steps {
                sshagent(['gitlab-ssh']) {
                    sh 'git restore v.txt'
                    sh "git tag ${version}"
                    sh "git push origin ${version}"
                }
            }
        }
  }
}
