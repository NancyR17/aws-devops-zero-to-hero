# AWS Continuous Integration Demo

## Set Up GitHub Repository

The first step in our CI journey is to set up a GitHub repository to store our Python application's source code. If you already have a repository, feel free to skip this step. Otherwise, let's create a new repository on GitHub by following these steps:

- Go to github.com and sign in to your account.
- Click on the "+" button in the top-right corner and select "New repository."
- Give your repository a name and an optional description.
- Choose the appropriate visibility option based on your needs.
- Initialize the repository with a README file.
- Click on the "Create repository" button to create your new GitHub repository.

Great! Now that we have our repository set up, we can move on to the next step.

## Create an AWS CodePipeline
In this step, we'll create an AWS CodePipeline to automate the continuous integration process for our Python application. AWS CodePipeline will orchestrate the flow of changes from our GitHub repository to the deployment of our application. Let's go ahead and set it up:
It will act as a Orchestrator and invoke AWS CodeBuild. When someone make changes to your code in GITHUB, it will call AWS codepipline and it will auto call to AWS CodeBuild automatically, we don't need to do it manually.

- Go to the AWS Management Console and navigate to the AWS CodePipeline service.
- Click on the "Create pipeline" button.
- Provide a name for your pipeline and click on the "Next" button.
- Choose Role: AWSCodePipeline[Tick Yes ]
- For the source stage, select "GitHub" as the source provider.
- Connect your GitHub account to AWS CodePipeline and select your repository.
- Choose the REPOSTORY NAMEbranch you want to use for your pipeline., outoputfromat: deafult CodePipeline
- In the build stage, select "AWS CodeBuild" as the build provider.
- Create a new CodeBuild project by clicking on the "Create project" button., 
- Configure the CodeBuild project with the necessary settings for your Python application, such as the build environment,  build commands, and artifacts.
- Save the CodeBuild project and go back to CodePipeline.
- Continue configuring the pipeline stages, such as deploying your application using AWS Elastic Beanstalk or any other suitable deployment option.
- Review the pipeline configuration and click on the "Create pipeline" button to create your AWS CodePipeline.

Awesome job! We now have our pipeline ready to roll. Let's move on to the next step to set up AWS CodeBuild.

## Configure AWS CodeBuild

In this step, we'll configure AWS CodeBuild to build our Python application based on the specifications we define. CodeBuild will take care of building and packaging our application for deployment. Follow these steps:

- In the AWS Management Console, navigate to the AWS CodeBuild service.
- Click on the "Create build project" button.
- Provide a name for your build project.
- For the source provider, choose "AWS CodePipeline."
- Select the pipeline you created in the previous step.
- Configure the build environment, such as the operating system, runtime, and compute resources required for your Python application.
- Assign the role as we are assiging roles to services ( code-build-f-role ) to aws codebuild.
- Go to IAM, create service role for service: codebuild, name it
- Specify the build commands, such as installing dependencies and running tests. Customize this based on your application's requirements.
- Runtime: Standard, Image: standard/latest, image version: latest version of runtime, Environemnt: : Linux, Privilgede ( Tick Yes ] > update Environment. 
- Go to AWs Systems Manager  > Parameter store > name it as standard: /app/docker-credentials/username , Type: Secure string , KMS Source: My current account, KMS Key ID: alias/aws/ssm, Value: value of dokcer username
- Create same for password, docker-registry/url with value as docker.io
- Now you have created the role but you havn't provided the access to this service,  if it was iam user, then we will got to IAM user, but now this is IAM service, we will go to IAM roles and grant the permissions.
- Go to IAM > Roles> go to ur role created ( code-build-service-f-role) > add permission > attach policies > search for ssm > AmazonSSMFullAccess( beacuse this is demo) > grant access.
  Error: Cannot connect to Docker daemon at unix://var/run/docker.sock Is the Docker daemon running?
- Now grant additional permissions to the CodeBuild. as bydeafult AWS CodeBuild does not allow you to create Docker imge. 
- Developer Tools > CodeBuild > Build Projects > sample-python-flask-service > Edit environemnt > Privilgede ( Tick Yes ] 
- Set up the artifacts configuration to generate the build output required for deployment.
- Review the build project settings and click on the "Create build project" button to create your AWS CodeBuild project.

Fantastic! With AWS CodeBuild all set up, we're now ready to witness the magic of continuous integration in action.

## Trigger the CI Process

In this final step, we'll trigger the CI process by making a change to our GitHub repository. Let's see how it works:

- Go to your GitHub repository and make a change to your Python application's source code. It could be a bug fix, a new feature, or any other change you want to introduce.
- Commit and push your changes to the branch configured in your AWS CodePipeline.
- Head over to the AWS CodePipeline console and navigate to your pipeline.
- You should see the pipeline automatically kick off as soon as it detects the changes in your repository.
- Sit back and relax while AWS CodePipeline takes care of the rest. It will fetch the latest code, trigger the build process with AWS CodeBuild, and deploy the application if you configured the deployment stage.

  ## Create an AWS CodeDeploy
- Go to Developer Tools > CodeDeoloy > Deployments  > Create Application
- Application name: sample-python-flask-app ( in Org-lets say payments-dev-amazon.com )
- Platform: EC2
  Launch Instances: Ec2, auto-assign public Ip: true, enable ssh 22 > create EC2.
- Go to instance settings > Manage Tags
  key: Project, value: payments > Savem Use same tag for this Project for every resource.
- Now we have to install a agent in ur ec2 instances,if there 10 ec2, then u have to do it in all EC2.
- Login to Ec2 in Putty or in WSL.
- ssh -i ~/Downloads/aws_login.pem ubuntu@ip, yes
- To install the CodeDeploy agent on Ubuntu Server
Sign in to the instance.
Enter the following commands, one after the other:
sudo apt update
sudo apt install ruby-full
sudo apt install wget
Enter the following command:
cd /home/ubuntu
/home/ubuntu represents the default user name for an Ubuntu Server instance. If your instance was created using a custom AMI, the AMI owner might have specified a different default user name.

Enter the following command:
wget https://bucket-name.s3.region-identifier.amazonaws.com/latest/install
bucket-name is the name of the Amazon S3 bucket that contains the CodeDeploy Resource Kit files for your region, and region-identifier is the identifier for your region.

For example:
https://aws-codedeploy-us-east-2.s3.us-east-2.amazonaws.com/latest/install

For a list of bucket names and region identifiers, see Resource kit bucket names by Region.

Enter the following command:
chmod +x ./install

Do one of the following:
To install the latest version of the CodeDeploy agent on any supported version of Ubuntu Server except 20.04:
sudo ./install auto - DO THIS ONLY, NOT ALL COMMANDS

To install the latest version of the CodeDeploy agent on Ubuntu Server 20.04:
Note
Writing the output to a temporary log file is a workaround that should be used while we address a known bug with the install script on Ubuntu Server 20.04.
sudo ./install auto > /tmp/logfile

To install a specific version of the CodeDeploy agent on any supported version of Ubuntu Server except 20.04:
List the available versions in your region:
aws s3 ls s3://aws-codedeploy-region-identifier/releases/ --region region-identifier | grep '\.deb$'

Install one of the versions:
sudo ./install auto -v releases/codedeploy-agent-###.deb
Note
AWS supports the latest minor version of the CodeDeploy agent. Currently the latest minor version is 1.7.x.

To install a specific version of the CodeDeploy agent on Ubuntu Server 20.04:
List the available versions in your region:
aws s3 ls s3://aws-codedeploy-region-identifier/releases/ --region region-identifier | grep '\.deb$'

Install one of the versions:
sudo ./install auto -v releases/codedeploy-agent-###.deb > /tmp/logfile
Note
Writing the output to a temporary log file is a workaround that should be used while we address a known bug with the install script on Ubuntu Server 20.04.
Note
AWS supports the latest minor version of the CodeDeploy agent. Currently the latest minor version is 1.7.x.

To check that the service is running
Enter the following command:
systemctl status codedeploy-agent

If the CodeDeploy agent is installed and running, you should see a message like The AWS CodeDeploy agent is running.
If you see a message like error: No AWS CodeDeploy agent running, start the service and run the following two commands, one at a time:
systemctl start codedeploy-agent
systemctl status codedeploy-agent 

- Now, give permissions to your EC2 instance. Ec2 is going to talk to AWS Codedeploy, similarly Codedeploy is also going to
  talk to Codedeploy.
- Create a Role   & grant this role with Ec2 to talk to the codedeploy.
  When you are granting permissions to Users to access any services on AWS - then we use IAM Users.
  When are granting permissions to any service to access any other service in AWS- then we use IAM Roles.
  So, Roles - is for Services , AND
  Users - is for Users like us.
- Go to IAM > Roles > create > Ec2-code-deploy-role > AWS services > services : Codedeploy > Next > Provide name: EC2-codedeploy-role > Create Role
- Assign this Role to EC2.  Go to EC2 > Actions > Security > Modify IAM Role > choose that role from dropdown ( code-deploy-role ) > Update
- After updating IAM, go to terminal & restart the agent service.
- sudo service codedeploy-agent restart
- In the Codedeploy, lets provide this EC2 instance as an agent, till now we have configured it, but haven't setup any connections to it.
- Create Deployment Target Groups, Go to Codedeploy > application > deployment groups > create > Name: sample-python-app, service role: use same service role ( existing codedeploy access, modify ( EC2Full Access) & grant it the EC2 access as well ).or u can use 2 different service roles. > Environemnt : EC2, key: value ( Name: sample-python ), Deployment settings: CodedeployDefault.AllAtOnce > Create deployment Group
- Go to DEployments > Create Deployment > Deployment-group: sample-python-app, My application is stored on GitHub  > connec to Git Hub > Repo Name: iam-veeramalla/aws-devops-zero-to-hero > commit ID: ( go to latest commit & put here ) > Create Deployment
Note: In CodeDeploy, you have to put the appspec.yaml in root of the directory, this is not the case with CodeBuild.
- 



  
