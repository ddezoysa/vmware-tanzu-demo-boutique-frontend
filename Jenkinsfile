pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh ''' echo "Running the build."
                export APP_NAME=${JOB_NAME%/*}
                export APP_VERSION=$(grep version ./frontend/Chart.yaml | awk \'{print $2}\' | sed \'s/\\"//g\')
                echo ${APP_NAME} > appNameVar
                echo ${APP_VERSION} > appVerVar
                '''
                appNameVar = readFile('appNameVar').trim()
                appVerVar = readFile('appVerVar').trim()

                echo "${appNameVar}"
                echo "${appVerVar}"
            }
        }
        stage('Build & Package') {
            steps {
                echo "${appNameVar}"
                echo "${appVerVar}"
                docker.build("${appNameVar}:${appVerVar}",".")
                docker.withRegistry('https://368696334230.dkr.ecr.ap-south-1.amazonaws.com','ecr:ap-south-1:awsECRDev')
                {
                    docker.image("${appNameVar}:${appVerVar}").push("${appVerVar}")
                }
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                export APP_NAME=${JOB_NAME%/*}
                export APP_VERSION=$(grep version ./frontend/Chart.yaml | awk \'{print $2}\' | sed \'s/\\"//g\')
                export NAMESPACE=dev
                export ENVIRONMENT=dev
                if helm ls --namespace ${NAMESPACE} |grep ${APP_NAME}; then
                  echo "Release ${APP_NAME} exists, Upgrading !! "
                  helm del --purge $APP_NAME
                  helm install -n ${APP_NAME} --namespace ${NAMESPACE} -f ${APP_NAME}/env/${ENVIRONMENT}-values.yaml --version ${APP_VERSION} ${APP_NAME}/
                  # helm upgrade --namespace ${NAMESPACE} -f ${APP_NAME}/env/${ENVIRONMENT}-values.yaml ${APP_NAME} ${APP_NAME}/
                else
                  echo "Release ${APP_NAME} not found, Installing!!"
                  helm install -n ${APP_NAME} --namespace ${NAMESPACE} -f ${APP_NAME}/env/${ENVIRONMENT}-values.yaml ${APP_NAME}/
                fi
                '''
            }
        }
    }
}