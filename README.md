To set up a CI/CD pipeline on AWS integrated with GitHub, we'll be using AWS CodePipeline, AWS CodeBuild, and GitHub. Here's a detailed, step-by-step guide on how to do it:

### Prerequisites:
1. **AWS Account**: Make sure you have an active AWS account.
2. **GitHub Account**: You'll need a GitHub repository.
3. **IAM Role for CodePipeline**: Ensure you have the necessary permissions in your AWS account to create services like CodePipeline, CodeBuild, and IAM roles.

---

### Step 1: **Create a GitHub Repository**
- Go to [GitHub](https://github.com/) and log in.
- Create a new repository. Name it something like `aws-cicd-demo` (or your choice).
- Add some sample code, for example, a simple HTML or Node.js project.

### Step 2: **Create an S3 Bucket for Artifacts**
1. Go to the AWS Management Console.
2. Navigate to **S3**.
3. Click **Create bucket**.
4. Give your bucket a unique name (e.g., `my-cicd-artifacts`) and click **Create**.
   - This bucket will store the artifacts (build results).

### Step 3: **Set up an IAM Role for CodePipeline**
1. Go to the AWS Management Console and navigate to **IAM**.
2. Click **Roles** on the left side.
3. Click **Create role**.
4. Choose **AWS Service** and select **CodePipeline**.
5. Attach the following policies:
   - **AWSCodePipelineFullAccess**
   - **AWSCodeBuildAdminAccess**
   - **AmazonS3FullAccess**
6. Click **Next** and name the role `CodePipelineServiceRole`.

---
If you don’t see **CodePipeline** as an option when creating an IAM role, follow these steps to resolve the issue:

### Manually Attach Policies to a Custom Role
1. **Create a custom IAM role**:
   - Go to the **IAM** console.
   - Select **Roles** > **Create Role**.
   - Choose **AWS Service** as the trusted entity type.
   - Select **EC2** or any service as a placeholder (you’ll adjust this later).
   - Click **Next** to attach permissions.

2. **Manually attach CodePipeline permissions**:
   - After creating the role, go to **IAM** > **Roles** and find the role you just created.
   - Select the role and click **Attach policies**.
   - Search for the following managed policies:
     - **AWSCodePipelineFullAccess**
     - **AWSCodeBuildAdminAccess**
     - **AmazonS3FullAccess** (if you need to access S3 buckets).
     - **AWSCodeDeployFullAccess** (if you use CodeDeploy in your pipeline).
   - Attach these policies to your role.

3. **Modify the trust relationship**:
   - In the IAM role settings, click **Trust relationships** > **Edit trust relationship**.
   - Replace the service that was initially selected (e.g., EC2) with **CodePipeline**. Use this trust relationship:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "codepipeline.amazonaws.com"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

4. **Update your CodePipeline with the new role**:
   - Now you can use this custom role in your CodePipeline creation.
---

### Step 4: **Create an IAM Role for CodeBuild**
1. In the IAM console, create a new role for **CodeBuild**.
2. Attach the following policies:
   - **AWSCodeBuildAdminAccess**
   - **AmazonS3FullAccess**
   - **CloudWatchLogsFullAccess**
3. Name the role `CodeBuildServiceRole`.

---

### Step 5: **Create a CodePipeline**
1. Go to the **CodePipeline** service in the AWS console.
2. Click **Create Pipeline**.
3. **Pipeline Name**: Enter a name for your pipeline (e.g., `MyGitHubPipeline`).
4. **Service Role**: Select the role you created for CodePipeline.
5. **Artifact Store**: Select the S3 bucket you created earlier.

---

### Step 6: **Add Source Stage (GitHub)**
1. In the source provider section, select **GitHub (Version 2)**.
2. Click **Connect to GitHub** and authorize AWS to access your GitHub account.
3. Choose your repository and branch (e.g., `main`).
4. Click **Next** to proceed to the next step.

---

### Step 7: **Add Build Stage (CodeBuild)**
1. In the **Add Build Stage**, select **AWS CodeBuild** as the build provider.
2. Click **Create a new build project**.
3. **Project Name**: Give your project a name (e.g., `MyBuildProject`).
4. **Environment**: Choose a managed image for your build. For example:
   - **Runtime**: Ubuntu
   - **Version**: Choose `aws/codebuild/standard:5.0`.
   - **Buildspec**: In this example, let’s use a simple `buildspec.yml` to define build steps.

Here is a sample `buildspec.yml` you can put in the root of your GitHub repository:

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo Installing dependencies...
      - npm install
  build:
    commands:
      - echo Build started on `date`
      - echo Compiling the application...
      - npm run build
artifacts:
  files:
    - '**/*'
```

5. For the **Service Role**, select the IAM role you created for CodeBuild.
6. Click **Create build project**.

---

### Step 8: **Deploy Stage (Optional)**
- If you want to deploy the application after building, you can add another stage to deploy to services like **Elastic Beanstalk**, **ECS**, or **S3**.
- Example for S3:
  - Create an additional stage in CodePipeline.
  - Choose **AWS S3** as the deploy action.
  - Specify the bucket you want to deploy your application to.

---

### Step 9: **Review and Create the Pipeline**
1. Review all the stages.
2. Click **Create Pipeline**.

Your pipeline will automatically trigger the build whenever a new change is pushed to the GitHub repository.

---

### Step 10: **Test Your Pipeline**
1. Go to your GitHub repository and make a change (like updating a README file).
2. Commit and push the changes.
3. Go back to your AWS CodePipeline and watch the pipeline run.

---

### Optional Additions
- **Notifications**: You can set up Amazon SNS or CloudWatch Events to get notifications on pipeline execution results.
- **CodeDeploy Integration**: Add CodeDeploy to automate deployments to EC2 or Lambda.

This setup will automatically build your project whenever you push new changes to GitHub and deploy them to your AWS environment. Let me know if you need further details!
