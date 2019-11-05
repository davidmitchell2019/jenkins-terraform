pipeline {  
    agent any  
    environment {
        /*envs for Terraform, will be jenkins secrets*/
        ARM_CLIENT_ID=""
        ARM_CLIENT_SECRET=""
        ARM_SUBSCRIPTION_ID=""
        ARM_TENANT_ID=""
        /*envs for inspec, will be jenkins secrets*/
        AZURE_SUBSCRIPTION_ID=""
        AZURE_CLIENT_ID=""
        AZURE_CLIENT_SECRET=""
        AZURE_TENANT_ID=""
        /*terraform vars*/
        DATABASE_USER="postgresuser"
        DATABASE_PASSWORD=""
        ACTION="destroy"
    }
    stages { 
      stage('Azure login') {
        when{
                equals expected:"apply",actual: "$ACTION"
        }
        steps {
            sh 'az login --service-principal --username $ARM_CLIENT_ID --password $ARM_CLIENT_SECRET --tenant $ARM_TENANT_ID'
        }
      }
      stage('Checkout') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
          git(
            url: 'https://github.com/davidmitchell2019/azuresql.git',
            credentialsId: 'gitcreds',
            branch: 'master'
            )
        }
      }
      stage('TF Init') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
           sh 'terraform init'
        }
      }
      stage('TF Validate') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
           sh 'terraform validate || rm -r -f *'
        }
      }
      stage('TF Plan') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
           sh 'terraform plan -out myplan -var database-login=$DATABASE_USER -var database-password=$DATABASE_PASSWORD || rm -r -f *'
        }
      }
      stage('Approval') {
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
            sh 'terraform apply -auto-approve -var database-login=$DATABASE_USER -var database-password=$DATABASE_PASSWORD || rm -f -r *'
        }
      }
      stage('Inspec validate') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            sh 'inspec exec /var/lib/jenkins/workspace/terraform-azure-postgres/inspec/ -t azure:// --chef-license accept-silent || terraform destroy -var database-login=$DATABASE_USER -var database-password=$DATABASE_PASSWORD -auto-approve '
        }
      }
      stage('Azure logout') {
        when{
                equals expected:"apply",actual: "$ACTION"
            }
        steps {
            sh 'az logout'
        }
      }
      stage('Destroy') {
        when{
                equals expected:"destroy",actual: "$ACTION"
            }
        steps {
            sh 'terraform destroy -auto-approve -var database-login=$DATABASE_USER -var database-password=$DATABASE_PASSWORD'
        }
      }
    }
}
