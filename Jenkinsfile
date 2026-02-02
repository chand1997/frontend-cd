pipeline{
    agent { label "agent-1" }
    parameters{
        string(name: 'version', description: 'Enter version')
    }
    stages{
    stage("get-target-arn"){
        steps{
            script{
                withAWS(credentials: 'aws-creds'){
                    env.TARGET_ARN = sh(
                    script: """
                            aws elbv2 describe-target-groups \
                             --names expense-dev \
                             --query 'TargetGroups[0].TargetGroupArn' \
                             --output text
                            
                            """,
                            returnStdout: true
                    ).trim()
                }
            }
        }
    }
    stage("frontend-cd"){
       steps{
        script{
            withAWS(credentials: 'aws-creds'){
            sh """
                aws eks update-kubeconfig --region us-east-1 --name expense-dev
                sed -i 's/IMAGE_VERSION/${params.version}/g' values.yaml
                sed -i 's|TARGET_ARN|${env.TARGET_ARN}|g' values.yaml
                helm upgrade --install frontend -n expense -f values.yaml .
                """
            }  
        }
       }
    }

    }
   

}
