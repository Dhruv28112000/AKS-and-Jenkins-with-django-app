pipeline {
    agent any
    environment{
        SUB_ID='f724e338-e3f9-4799-b561-afb0c8d99728'
        TENANT_ID='fb74c11b-dc74-4e33-a372-4b2fd6e583ec'
        ACR_NAME='demoacrtml'
        REPO='jenkins_demo'
        IMG_NAME='django_v1'
        TAG='latest'
        RG='Dhruv-RG'
        AKS_NAME='myaks-django'
        ACR_ID='/subscriptions/f724e338-e3f9-4799-b561-afb0c8d99728/resourceGroups/Dhruv-RG/providers/Microsoft.ContainerRegistry/registries/demoacrtml'
    }
    stages {
        stage('Clone') {
            steps {
                dir('Code-repo') {
                echo 'Cloning the GITHUB Code'
                git url:"https://github.com/Dhruv28112000/AKS-and-Jenkins-with-django-app", branch: "main"
                }
            }
        }
        stage('Build') {
            steps {
                echo 'Login to Azure account'
                script{
                    dir('Code-repo') {
                        withCredentials([usernamePassword(credentialsId: 'AZ_SVC_ID', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                        sh """
                         az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $TENANT_ID
                         az account set -s $SUB_ID
                         az acr login --name $ACR_NAME --resource-group $RG
                         az acr build --image $REPO/$IMG_NAME:$tag --registry $ACR_NAME --file Dockerfile .
                         az role assignment create --assignee $AZURE_CLIENT_ID --scope $ACR_ID --role acrpull
                         """
                        }
                    }
                }
            }
        }
        stage('Push') {
            steps {
                echo 'Deploying code to AKS'
                dir('Code-repo'){
                withCredentials([usernamePassword(credentialsId: 'AZ_SVC_ID', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                script{
                    sh """
                    kubectl create secret docker-registry demosecret --namespace default --docker-server=demoacrtml.azurecr.io --docker-username=$AZURE_CLIENT_ID --docker-password=$AZURE_CLIENT_SECRET
                    az aks get-credentials --resource-group $RG --name $AKS_NAME
                    kubectl apply -f pod.yaml
                    """
                }
                }
                }
            }
        }
}
}

