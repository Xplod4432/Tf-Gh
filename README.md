# Integration and Testing Document

## GitHub Actions for Terraform

## Abstract
Terraform init and validate is applied on each Pull Request. This helps automate the workflow and tasks that take place between writing the scripts and the changes being made to AWS Resources. This will be achieved using GitHub Actions which perform an action whenever it is triggered using various actions such as a push, commit or a pull request.

## Prerequisites
- [ ] Root or IAM User Access Token
- [ ] GitHub Account
- [ ] Basic Git branching knowledge

## Generate Access Token
1. If you are using a root account it is recommended to **create an IAM user account** with the required permissions to create and edit resources.
2. After signing in on the top right corner click on your account name and select **Security credentials**.
<IMG1>
3. Scroll down to Access keys and click on create access key.
4. **Note**: The access key secret is only shown once so be sure to download it as a CSV and store it safely.
5. **Activate** the Access key
  
## Setting up the GitHub repository
1. Log into or create an account on **GitHub**.
2. Create a **new repository** (Visibility does not affect operations/workflow)
3. Go to **Settings** of the repo.
4. Select **Actions** under the **Secrets and variables** section
<IMG2>
5. Click on **New Repository secret**.
6. Add two secrets namely **AWS_ACCESS_KEY_ID** and **AWS_SECRET_ACCESS_KEY** with their respective values as obtained from the previous section.

## Workflow file
1. Create a folder named **.github**
2. Create a folder named **workflows** in the .github folder
3. Create a file named **tfint.yml** with the following code:

```
name: terraform-automation-action
run-name: Xplod4432 is learning GitHub Actions
on:
  push:
   branches:
   - main
   paths:
   - terraform/**
  pull_request:
    # Sequence of patterns matched against refs/heads
    branches:    
      - main
      - 'releases/**'

env:
 # verbosity setting for Terraform logs
 TF_LOG: INFO
 # Credentials for deployment to AWS
 AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
 AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
 
jobs:
 terraform:
   permissions: write-all
   name: "Terraform Infrastructure Change Management"
   runs-on: ubuntu-latest
   defaults:
     run:
       shell: bash
       # We keep Terraform files in the terraform directory.
       working-directory: ./terraform
 
   steps:
     - name: Checkout the repository to the runner
       uses: actions/checkout@v2
 
     - name: Setup Terraform with specified version on the runner
       uses: hashicorp/setup-terraform@v2
#        with:
#          terraform_version: 1.3.0
    
     - name: Terraform init
       id: init
       run: terraform init
 
#      - name: Terraform format
#        id: fmt
#        run: terraform fmt -check
    
     - name: Terraform validate
       id: validate
       run: terraform validate
 
     - name: Terraform plan
       id: plan
       if: github.event_name == 'pull_request'
       run: terraform plan -no-color -input=false
       continue-on-error: true
    
     - uses: actions/github-script@v6
       if: github.event_name == 'pull_request'
       env:
         PLAN: "terraform\n${{ steps.plan.outputs.stdout }}"
       with:
         script: |
           const output = `#### Terraform Initialization ⚙️\`${{ steps.init.outcome }}\`
           #### Terraform Validation 🤖\`${{ steps.validate.outcome }}\`
           #### Terraform Plan 📖\`${{ steps.plan.outcome }}\`
 
           <details><summary>Show Plan</summary>
 
           \`\`\`\n
           ${process.env.PLAN}
           \`\`\`
 
           </details>
           *Pushed by: @${{ github.actor }}, Action: \`${{ github.event_name }}\`*`;
 
           github.rest.issues.createComment({
             issue_number: context.issue.number,
             owner: context.repo.owner,
             repo: context.repo.repo,
             body: output
           })
 
     - name: Terraform Plan Status
       if: steps.plan.outcome == 'failure'
       run: exit 1
 
     - name: Terraform Apply
       if: github.ref == 'refs/heads/main' && github.event_name == 'push'
       run: terraform apply -auto-approve -input=false
```

## Working
1. This workflow acknowledges and performs actions for all the files present in the **terraform** folder in the root directory of repo.
2. This **terraform folder contains all the terraform scripts**.
3. Any **changes** to this folder, **commits**, **push** and **prs** will trigger the workflow.

## Testing
1. **Fork** the https://github.com/Xplod4432/Tf-Gh repo to the local machine or **clone** your own repo **with the relevant workflow files**.
2. Make changes to the script inside the terraform folder or simply change the readme file and **initiate a Pull Request**. 
3. Once the Pull Request is initiated, the preliminary checks and **Terraform validate will be performed**.
4. Once the **PR is merged only then Terraform Apply will be done** and the changes can be seen.

## Sources
[Repo Link](https://github.com/Xplod4432/Tf-Gh)

[Terraform Blog 1](https://spacelift.io/blog/github-actions-terraform)

[Terraform Blog 2](https://gaunacode.com/deploying-terraform-at-scale-with-github-actions)
