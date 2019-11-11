pipeline {  
    agent any  
    environment {
        /*envs for Terraform*/
        ARM_CLIENT_ID=""
        ARM_CLIENT_SECRET=""
        ARM_SUBSCRIPTION_ID=""
        ARM_TENANT_ID=""
        /*envs for inspec*/
        AZURE_SUBSCRIPTION_ID=""
        AZURE_CLIENT_ID=""
        AZURE_CLIENT_SECRET=""
        AZURE_TENANT_ID=""
        /*terraform vars*/
        DATABASE_USER="postgresuser"
        DATABASE_PASSWORD=""
        ACTION="apply" /*apply or destroy*/
        RESOURCE_GROUP=""
        SERVER_NAME=""
        /*Azure vars*/
        AZURESTORAGEACCOUNT=""
        AZURESTORAGECONTAINER=""
        }
    stages { 
      stage('Azure login') {
        steps {
            sh 'az login --service-principal --username $ARM_CLIENT_ID --password $ARM_CLIENT_SECRET --tenant $ARM_TENANT_ID'
        }
      }
      stage('Create Azure Resource group') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            sh 'az group create --location UKSouth --name $RESOURCE_GROUP || echo "rg already created"'
        }
      }
      stage('Create Azure storage account') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            sh 'az storage account create --name $AZURESTORAGEACCOUNT --resource-group $RESOURCE_GROUP --location UKSouth || echo "account already created"'
        }
      }
      stage('Create Azure storage container') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            sh 'az storage container create --name $AZURESTORAGECONTAINER --connection-string "$(az storage account show-connection-string -n $AZURESTORAGEACCOUNT -g $RESOURCE_GROUP --key primary)" || echo "container already created"'
        }
      }
      stage('Checkout') {
        steps {
          git(
            url: 'https://github.com/davidmitchell2019/azuresql.git',
            credentialsId: 'gitcreds',
            branch: 'master'
            )
        }
      }
      stage('TF Init') {
        steps {
           sh 'terraform init -backend-config="resource_group_name=$RESOURCE_GROUP" -backend-config="storage_account_name=$AZURESTORAGEACCOUNT" -backend-config="container_name=$AZURESTORAGECONTAINER" '
        }
      }
      stage('TF Validate') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
           sh 'terraform validate'
        }
      }
      stage('TF Plan') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
           sh 'terraform plan -var database-login=$DATABASE_USER -var database-password=$DATABASE_PASSWORD -var resource_group_name=$RESOURCE_GROUP -var storage_account_name=$AZURESTORAGEACCOUNT -var storage_container_name=$AZURESTORAGECONTAINER'
        }
      }
      stage('Approve apply') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
          script {
            def userInput = input(id: 'confirm', message: 'Apply Terraform?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Apply terraform', name: 'confirm'] ])
          }
        }
      }
      stage('TF Apply') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            sh 'terraform apply -auto-approve -var database-login=$DATABASE_USER -var database-password=$DATABASE_PASSWORD -var resource_group_name=$RESOURCE_GROUP -var storage_account_name=$AZURESTORAGEACCOUNT -var storage_container_name=$AZURESTORAGECONTAINER'
        }
      }
      stage('Inspec validate') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            sh 'inspec exec inspec -t azure:// --input resource_group=$RESOURCE_GROUP server_name=$SERVER_NAME --chef-license accept-silent'
        }
      }
      stage('Approve destroy') {
        when{
                equals expected:"destroy",actual: "$ACTION"
            }
        steps {
          script {
            def userInput = input(id: 'confirm', message: 'Destroy Terraform?', parameters: [ [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Apply terraform', name: 'confirm'] ])
          }
        }
      }
      stage('Destroy') {
        when{
                equals expected:"destroy",actual: "$ACTION"
            }
        steps {
            sh 'terraform destroy -auto-approve -force -var database-login=$DATABASE_USER -var database-password=$DATABASE_PASSWORD -var resource_group_name=$RESOURCE_GROUP -var storage_account_name=$AZURESTORAGEACCOUNT -var storage_container_name=$AZURESTORAGECONTAINER'
        }
      }
    }
    post {
        always {
            sh 'az logout || echo "logged out"'
            deleteDir() /* clean up our workspace */
        }
    }
}
