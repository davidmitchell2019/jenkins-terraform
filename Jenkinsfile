pipeline {  
    agent any  
    environment {
        /*envs for Terraform to connect to Azure*/
        ARM_CLIENT_ID=""
        ARM_CLIENT_SECRET=""
        ARM_SUBSCRIPTION_ID=""
        ARM_TENANT_ID=""
        /*envs for inspec to connect to Azure*/
        AZURE_SUBSCRIPTION_ID=""
        AZURE_CLIENT_ID=""
        AZURE_CLIENT_SECRET=""
        AZURE_TENANT_ID=""
        /*terraform vars*/
        DATABASE_USER="postgresuser"
        DATABASE_PASSWORD=""
        RESOURCE_GROUP=""
        SERVER_NAME=""
        STORAGEMB="5120"
        GEO_REDUNDENT_ENABLED="Disabled"
        BACKUP_RETENTION_DAYS="7"
        OFFICE_IP=""
        VNET_IP_CIDR="10.10.10.0/24"
        SUBNET_IP_CIDR="10.10.10.0/24"
        /*Terraform action destroy or apply*/
        ACTION="apply"
        /*Azure CLI vars*/
        AZURESTORAGEACCOUNT=""
        AZURESTORAGECONTAINER=""
        }
    stages { 
      stage('Azure login') {
        steps {
            /*Logs into azure*/
            sh 'az login --service-principal --username $ARM_CLIENT_ID --password $ARM_CLIENT_SECRET --tenant $ARM_TENANT_ID'
        }
      }
      stage('Create Azure Resource group') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            /*Creates a resource group in Azure when the action equals apply*/
            sh 'az group create --location UKSouth --name $RESOURCE_GROUP || echo "rg already created"'
        }
      }
      stage('Create Azure storage account') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            /*Creates an Azure storage account when the action equals apply*/
            sh 'az storage account create --name $AZURESTORAGEACCOUNT --resource-group $RESOURCE_GROUP --location UKSouth || echo "account already created"'
        }
      }
      stage('Create Azure storage container') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            /*Creates an Azure storage container when the action equals apply*/
            sh 'az storage container create --name $AZURESTORAGECONTAINER --connection-string "$(az storage account show-connection-string -n $AZURESTORAGEACCOUNT -g $RESOURCE_GROUP --key primary)" || echo "container already created"'
        }
      }
      stage('Checkout') {
        steps {
            /*clone from git repo using jenkins secret to store access token*/
          git(
            url: 'https://github.com/davidmitchell2019/azuresql.git',
            credentialsId: 'gitcreds',
            branch: 'master'
            )
        }
      }
      stage('TF Init') {
        steps {
            /*Initialize Terraform backend to use Azure blob storage*/
           sh 'terraform init -backend-config="resource_group_name=$RESOURCE_GROUP" -backend-config="storage_account_name=$AZURESTORAGEACCOUNT" -backend-config="container_name=$AZURESTORAGECONTAINER" '
        }
      }
      stage('TF Validate') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            /*Validate Terraform code*/
           sh 'terraform validate'
        }
      }
      stage('TF Plan') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            /*Terraform plan, to see what cxhanges are to be made*/
           sh 'terraform plan -out planfile -var database-login=$DATABASE_USER -var database-password=$DATABASE_PASSWORD -var resource_group_name=$RESOURCE_GROUP -var storage_account_name=$AZURESTORAGEACCOUNT -var storage_container_name=$AZURESTORAGECONTAINER -var postgres-server-name=$SERVER_NAME -var vnet_ip_cidr=$VNET_IP_CIDR -var subnet_ip_cidr=$SUBNET_IP_CIDR -var office_ip=$OFFICE_IP -var storagemb=$STORAGEMB -var backup_retention_days=$BACKUP_RETENTION_DAYS -var geo_redundent_enabled=$GEO_REDUNDENT_ENABLED'
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
            sh 'terraform apply -auto-approve planfile'
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
            sh 'terraform destroy -auto-approve -force -var database-login=$DATABASE_USER -var database-password=$DATABASE_PASSWORD -var resource_group_name=$RESOURCE_GROUP -var storage_account_name=$AZURESTORAGEACCOUNT -var storage_container_name=$AZURESTORAGECONTAINER -var postgres-server-name=$SERVER_NAME -var vnet_ip_cidr=$VNET_IP_CIDR -var subnet_ip_cidr=$SUBNET_IP_CIDR -var office_ip=$OFFICE_IP -var storagemb=$STORAGEMB -var backup_retention_days=$BACKUP_RETENTION_DAYS -var database-password=$DATABASE_PASSWORD -var resource_group_name=$RESOURCE_GROUP -var storage_account_name=$AZURESTORAGEACCOUNT -var storage_container_name=$AZURESTORAGECONTAINER -var postgres-server-name=$SERVER_NAME -var vnet_ip_cidr=$VNET_IP_CIDR -var subnet_ip_cidr=$SUBNET_IP_CIDR -var office_ip=$OFFICE_IP -var storagemb=$STORAGEMB -var backup_retention_days=$BACKUP_RETENTION_DAYS -var geo_redundent_enabled=$GEO_REDUNDENT_ENABLED'
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
