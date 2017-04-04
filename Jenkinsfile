node {
    checkout scm

    env.REPO_NAME = sh(
        script: "echo $env.JOB_NAME | cut -f 2 -d /",
        returnStdout: true
    ).trim()

    env.DOCKER_TAG = sh(
        script: "git rev-parse HEAD | cut -c1-8",
        returnStdout: true
    ).trim()

    env.DOCKER_REGISTRY = "593291632749.dkr.ecr.eu-west-1.amazonaws.com"

    stage('Configure Docker registry') {
        withCredentials([usernamePassword(credentialsId: 'aws-ecr',
                                          passwordVariable: 'AWS_ACCESS_KEY_ID',
                                          usernameVariable: 'AWS_SECRET_ACCESS_KEY')]) {
            sh """
            aws configure set default.region eu-west-1
            aws configure set aws_access_key_id $AWS_SECRET_ACCESS_KEY
            aws configure set aws_secret_access_key $AWS_ACCESS_KEY_ID

            if ! aws ecr describe-repositories --repository-names ${env.REPO_NAME} > /dev/null 2>&1; then
                aws ecr create-repository --repository-name ${env.REPO_NAME};
            fi
            """
        }
    }

    stage('Docker build') {
        sh """
        docker build \
            -t ${env.DOCKER_REGISTRY}/${env.REPO_NAME}:${env.DOCKER_TAG} \
            -t ${env.DOCKER_REGISTRY}/${env.REPO_NAME}:latest \
            .
        """
    }

    stage('Docker push') {
        sh "cat ~/.aws/credentials"
        sh "\$(aws ecr get-login)"
        sh "docker push ${env.DOCKER_REGISTRY}/${env.REPO_NAME}:${env.DOCKER_TAG}"
        sh "docker push ${env.DOCKER_REGISTRY}/${env.REPO_NAME}:latest"
    }

    stage('Deploy') {
        echo 'Deploying....'
    }
}
