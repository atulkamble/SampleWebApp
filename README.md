# SampleWebApp
A simple Node.js web server application demonstrating the use of AWS CodeCommit, CodeBuild, and CodeDeploy for continuous integration and continuous deployment (CI/CD) on an EC2 instance.

### **Tags**:
- Node.js
- AWS CodeCommit
- AWS CodeBuild
- AWS CodeDeploy
- CI/CD Pipeline
- EC2 Deployment
- DevOps

```
cd Downloads
ssh -i "terraformkey.pem" ec2-user@ec2-54-84-239-38.compute-1.amazonaws.com
```

### Prerequisites:
1. **AWS CLI** installed and configured. [Download and install AWS CLI](https://aws.amazon.com/cli/)
2. **IAM Role** with necessary permissions (CodeCommit, CodeBuild, CodeDeploy, EC2 access).
3. **EC2 Instance** running Amazon Linux 2 or another supported platform.
4. **AWS CodeDeploy agent** installed on the EC2 instance.
// Install the CodeDeploy agent for Amazon Linux or RHEL
```
sudo yum update -y
sudo yum install ruby -y
sudo yum install wget -y
wget https://aws-codedeploy-us-east-2.s3.us-east-2.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
systemctl status codedeploy-agent
systemctl start codedeploy-agent
```
---

### Step 1: Create a CodeCommit Repository

```
sudo yum install git -y
git --version
```
1. **Create Repository in CodeCommit:**
   - Open the AWS Management Console and go to **CodeCommit**.
   - Create a new repository named `SampleWebApp`.
   
2. **Clone the Repository Locally:**
   ```bash
   git clone https://git-codecommit.<region>.amazonaws.com/v1/repos/SampleWebApp
   cd SampleWebApp
   ```

### Step 2: Create a Simple Node.js Application
```
sudo yum install npm -y
npm --version
```
```
mkdir sample-web-app
cd sample-web-app
```
1. **Initialize a Node.js project:**
   ```bash
   npm init -y
   ```

2. **Install Express.js:**
   ```bash
   npm install express
   ```

3. **Create `app.js` file:**
   ```javascript
   const express = require('express');
   const app = express();

   app.get('/', (req, res) => {
       res.send('Hello from CodeDeploy!');
   });

   const port = process.env.PORT || 3000;
   app.listen(port, () => {
       console.log(`App is running on port ${port}`);
   });
   ```

4. **Create a `package.json` script to run the application:**
   Add this to your `package.json`:
   ```json
   "scripts": {
       "start": "node app.js"
   }
   ```

### Step 3: Create a BuildSpec File for CodeBuild

1. **Create `buildspec.yml`** in the root of your repository:

   ```
   touch buildspec.yml
   sudo nano buildspec.yml
   ```
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
         - npm run start
   ```

### Step 4: Create an AppSpec File for CodeDeploy

1. **Create a `appspec.yml`** in the root of your repository:

   ```
   touch appspec.yml
   sudo nano appspec.yml
   ```
   ```yaml
   version: 0.0
   os: linux
   files:
     - source: /
       destination: /home/ec2-user/sample-web-app

   hooks:
     AfterInstall:
       - location: scripts/install_dependencies.sh
         timeout: 300
         runas: ec2-user
     ApplicationStart:
       - location: scripts/start_server.sh
         timeout: 300
         runas: ec2-user
   ```

### Step 5: Create Shell Scripts for Deployment

1. **Create a directory `scripts/`** and add the following files:

   **`install_dependencies.sh`**
   ```
   touch install_dependencies.sh
   sudo nano install_dependencies.sh
   ```
   ```bash
   #!/bin/bash
   cd /home/ec2-user/sample-web-app
   npm install
   ```

   **`start_server.sh`**
   ```
   touch start_server.sh
   sudo nano start_server.sh
   ```
   ```bash
   #!/bin/bash
   cd /home/ec2-user/sample-web-app
   npm start &
   ```

3. **Make the scripts executable:**
   ```bash
   chmod +x scripts/*.sh
   ```

### Step 6: Push the Code to CodeCommit

1. **Add, commit, and push the code to CodeCommit:**
   ```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
   ```

### Step 7: Set Up AWS CodeBuild

1. **Create a CodeBuild Project:**
   - Go to **AWS CodeBuild** in the console.
   - Create a new build project, selecting the **CodeCommit** repository and configuring the build environment for **Node.js**.

2. **Choose BuildSpec:**
   - Choose the `buildspec.yml` file for the build commands.

### Step 8: Set Up AWS CodeDeploy

1. **Create an Application in CodeDeploy:**
   - Go to **AWS CodeDeploy** and create a new application.
   - Create a **deployment group** for your EC2 instances with the appropriate IAM role.

2. **Configure EC2 Instances:**
   Ensure the **CodeDeploy agent** is installed on your EC2 instance:
   ```bash
   sudo yum update -y
   sudo yum install -y ruby
   cd /home/ec2-user/
   wget https://bucket-name.s3.region.amazonaws.com/latest/install
   chmod +x ./install
   sudo ./install auto
   sudo service codedeploy-agent start
   ```

### Step 9: Set Up a Deployment Pipeline

1. **Create a Pipeline:**
   - Go to **AWS CodePipeline** and create a new pipeline.
   - Set **CodeCommit** as the source.
   - Set **CodeBuild** as the build stage.
   - Set **CodeDeploy** as the deploy stage.

2. **Deploy the Application:**
   - Push new changes to trigger the pipeline.

---

### Testing and Conclusion

After pushing your code and triggering the pipeline, CodeCommit will store the code, CodeBuild will build the application, and CodeDeploy will deploy it to your EC2 instance. You can visit your EC2 instance's public IP to see the running application.
