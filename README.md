# aws-ebeanstalk-py-django-01

AWS Elastic Beanstalk deployment of Python Django simple web application

**WARNING:** to AVOID incurring high cost on the AWS account, please
remember to delete all the AWS resources & services created by this project,
by following the CLEANUP steps provided below. 

## Prerequisite

- Python environment 
  - Note: linux (Ubuntu) comes with python pre-installed; for example,
    the following linux runs on Windows WSL
```
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.4 LTS
Release:        20.04
Codename:       focal

$ python3 --version
Python 3.8.10

$ pip --version
pip 20.0.2 from /usr/lib/python3/dist-packages/pip (python 3.8)
```
  - Note: one option for running linux on the Windows computer is to use WSL; 
    see [Install Linux on Windows with WSL](https://docs.microsoft.com/en-us/windows/wsl/install)

## Setup

- Clone this repository to get the artifacts that have been pre-created 
  so don't need to be recreated, including:
  - The scaffold Django web application `webapp` that was generated using
    the following in the python virtual environment. The virtual environment
    was not checkin with the repo, hence needs to be recreated.
    Note: see below for how to create the Python virtual environment.
```
$ cd [projects-dir]/aws-ebeanstalk-py-django-01
$ virtualenv ./venv
$ source ./venv/bin/activate
(venv) $ pip install django==2.2
(venv) $ django-admin startproject webapp
(venv) $ pip freeze > requirements.txt
```

- Create the Python virtual environment. The virtual environment is not included
  with this repo; it's ignored in the `.gitignore` file. To create the virtual
  environment:
```
$ cd [projects-dir]/aws-ebeanstalk-py-django-01
$ virtualenv ./venv
$ source ./venv/bin/activate
(venv) $ cd [projects-dir]/aws-ebeanstalk-py-django-01/webapp
(venv) $ pip install -r requirements.txt -v
```

- Verify the webapp
```
(venv) $ cd [projects-dir]/aws-ebeanstalk-py-django-01/webapp
(venv) $ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...
[...]
Django version 4.0.6, using settings 'webapp.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

## Develop

The Django web application is in the `webapp` folder. Feel free to make 
any adjustments.  

## Deploy

The webapp will be deployed to AWS using Elastic Beanstalk. The setup is as
follow:

- Prepare configuration for Elastic Beanstalk deployment.
  - Note: by default, Elastic Beanstalk looks for a file named application.py 
  to start your application. Because this doesn't exist in this Django project 
  we need to make some adjustments to the application's environment.
  We also set environment variables so that the application's modules can be 
  loaded.
  - Warning: the `.ebextensions` folder must be created in the `webapp` folder
```
$ cd [projects-dir]/aws-ebeanstalk-py-django-01/webapp
$ mkdir .ebextensions
$ touch ./.ebextensions/django.config
```

  - The configurations in django.config, `WSGIPath`, specifies the location of
    the WSGI script that Elastic Beanstalk uses to start the application.
    The path is relative to the `.ebextensions` folder.
```
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: webapp.wsgi:application
```

- We are using Elastic Beanstalk CLI to deploy. Alternatively, we can also use 
  the [Elastic Beanstalk console](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/GettingStarted.DeployApp.html).
  Install the EB CLI in a virtual environment; see REF-1 & REF-2 for in-depth 
  details.
```
$ cd [projects-dir]/aws-ebeanstalk-py-django-01
$ source ./venv/bin/activate
(venv) $ pip install awsebcli --upgrade
(venv) $ eb --version
EB CLI 3.20.3 (Python 3.8.1)
```

- Initialize your Elastic Beanstalk CLI repository. 
  - NOTE: you will need the `aws-access-id` and `aws-secret-key` which can be 
  obtained from the "AWS Console > IAM > Users" page, then select your username,
  that will direct to the "User Summary" page, then go to the
  "Security Credential" page. You will find the "Access keys" table that shows
  all the created access keys. Note: for each "access key" there is a corresponding
  "secret key". You should have the secret key saved somewhere because it can't
  be retrieved from the AWS console. It is only presented once during creation.

  - If you have used AWS CLI, and you may have stored the `aws-access-id` and 
    `aws-secret-key` in the `~/.aws/credentials` file. In this case, during the
     `eb init` below, the credential will not be requested again.    
```
(venv) $ cd [projects-dir]/aws-ebeanstalk-py-django-01/webapp
(venv) $ eb init -p python-3.8 aws-ebeanstalk-py-django-01
You have not yet set up your credentials or your credentials are incorrect
You must provide your credentials.
(aws-access-id): *******
(aws-secret-key): *******
Application aws-ebeanstalk-py-django-01 has been created.
```

- At this point, go to AWS console "Elastic Beanstalk > Applications" page, and
  the application `aws-ebeanstalk-py-django-01` should have been created.
  
- Next, create an environment and deploy your application to the environment. 
  The following command creates a load-balanced Elastic Beanstalk environment 
  named `aws-ebeanstalk-py-django-01-env`. Creating an environment takes about 
  5 minutes. As Elastic Beanstalk creates the resources needed to run your 
  application, it outputs informational messages that the EB CLI relays to your
  terminal.
```
(venv) $ eb create aws-ebeanstalk-py-django-01-env
Creating application version archive "app-4908-220729_132840938270".
Uploading aws-ebeanstalk-py-django-01/app-4908-220729_132840938270.zip to S3. This may take a while.
Upload Complete.
Environment details for: aws-ebeanstalk-py-django-01-env
  Application name: aws-ebeanstalk-py-django-01
  Region: us-west-2
  Deployed Version: app-4908-220729_132840938270
  Environment ID: e-vzdpqf7fcm
  Platform: arn:aws:elasticbeanstalk:us-west-2::platform/Python 3.8 running on 64bit Amazon Linux 2/3.3.15
  Tier: WebServer-Standard-1.0
  CNAME: UNKNOWN
  Updated: 2022-07-29 20:28:50.635000+00:00
Printing Status:
2022-07-29 20:28:49    INFO    createEnvironment is starting.
2022-07-29 20:28:50    INFO    Using elasticbeanstalk-us-west-2-349327579537 as Amazon S3 storage bucket for environment data.
2022-07-29 20:29:16    INFO    Created security group named: sg-0d4e57ecfe78dd7f3
2022-07-29 20:29:32    INFO    Created load balancer named: awseb-e-v-AWSEBLoa-11E3Z8HYHWI7M
2022-07-29 20:29:32    INFO    Created security group named: awseb-e-vzdpqf7fcm-stack-AWSEBSecurityGroup-14EBI3MI8MEZG
2022-07-29 20:29:32    INFO    Created Auto Scaling launch configuration named: awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingLaunchConfiguration-gdwq18IM1imP
 -- Events -- (safe to Ctrl+C)
2022-07-29 20:30:36    INFO    Created Auto Scaling group named: awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingGroup-O5BEPBX3UO1Z
2022-07-29 20:30:36    INFO    Waiting for EC2 instances to launch. This may take a few minutes.
2022-07-29 20:30:51    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:us-west-2:349327579537:scalingPolicy:76cf55ec-df94-4d28-9d6b-b7f488b0471f:autoScalingGroupName/awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingGroup-O5BEPBX3UO1Z:policyName/awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingScaleDownPolicy-VpH1cGs0NFvC
2022-07-29 20:30:51    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:us-west-2:349327579537:scalingPolicy:62397ed2-02a4-4dcd-bc57-d352ad65b780:autoScalingGroupName/awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingGroup-O5BEPBX3UO1Z:policyName/awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingScaleUpPolicy-QrGFM41rCbk4
2022-07-29 20:30:51    INFO    Created CloudWatch alarm named: awseb-e-vzdpqf7fcm-stack-AWSEBCloudwatchAlarmHigh-4AXGHSGTZFIQ
2022-07-29 20:30:52    INFO    Created CloudWatch alarm named: awseb-e-vzdpqf7fcm-stack-AWSEBCloudwatchAlarmLow-9TPSITYGQX56
2022-07-29 20:30:55    INFO    Instance deployment successfully generated a 'Procfile'.
2022-07-29 20:30:58    INFO    Instance deployment completed successfully.
2022-07-29 20:32:01    INFO    Successfully launched environment: aws-ebeanstalk-py-django-01-env
```

- When the environment creation process completes, find the domain name of 
  the newly created environment. The environment's domain name is the value of
  the `CNAME` property.
```
(venv) $ eb status
Environment details for: aws-ebeanstalk-py-django-01-env
  Application name: aws-ebeanstalk-py-django-01
  Region: us-west-2
  Deployed Version: app-220730_082255203723
  Environment ID: e-j2ybaiyamv
  Platform: arn:aws:elasticbeanstalk:us-west-2::platform/Python 3.8 running on 64bit Amazon Linux 2/3.3.15
  Tier: WebServer-Standard-1.0
  CNAME: aws-ebeanstalk-py-django-01-env.eba-djww4c8p.us-west-2.elasticbeanstalk.com
  Updated: 2022-07-30 15:25:50.379000+00:00
  Status: Ready
  Health: Green
```

- For security reason, 2-step deployment is used. Edit the Django's 
  configuration (`./webapp/webapp/settings.py`) to add the domain name that 
  Elastic Beanstalk assigned to the application to Django's ALLOWED_HOSTS. 
```
# File: ./webapp/webapp/settings.py

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

#ALLOWED_HOSTS = []
ALLOWED_HOSTS = ['aws-ebeanstalk-py-django-01-env.eba-h3hk2hdp.us-west-2.elasticbeanstalk.com']
```
  - Note: to test locally using `python manage.py runserver`, revert back to 
    empty `ALLOWED_HOSTS = []`

- Then, redeploy the application. This Django security requirement is designed 
  to prevent HTTP Host header attacks.
```
(venv) $ eb deploy
Creating application version archive "app-220730_093052548799".
Uploading aws-ebeanstalk-py-django-01/app-220730_093052548799.zip to S3. This may take a while.
Upload Complete.
2022-07-30 16:31:02    INFO    Environment update is starting.
2022-07-30 16:31:06    INFO    Deploying new version to instance(s).
2022-07-30 16:31:10    INFO    Instance deployment successfully generated a 'Procfile'.
2022-07-30 16:31:18    INFO    Instance deployment completed successfully.
2022-07-30 16:31:23    INFO    New application version was deployed to running EC2 instances.
2022-07-30 16:31:23    INFO    Environment update completed successfully.
```

- Open browser and point to the the domain name that Elastic Beanstalk assigned 
  to the application: `https://aws-ebeanstalk-py-django-01-env.eba-h3hk2hdp.us-west-2.elasticbeanstalk.com`

## CLEANUP

- Terminate your Elastic Beanstalk environment
```
(venv) $ eb terminate aws-ebeanstalk-py-django-01-env
The environment "aws-ebeanstalk-py-django-01-env" and all associated instances will be terminated.
To confirm, type the environment name: aws-ebeanstalk-py-django-01-env
2022-07-29 20:50:55    INFO    terminateEnvironment is starting.
2022-07-29 20:50:55    INFO    Validating environment's EC2 instances have termination protection disabled before performing termination.
2022-07-29 20:50:55    INFO    Finished validating environment's EC2 instances for termination protection.
2022-07-29 20:51:13    INFO    Deleted CloudWatch alarm named: awseb-e-vzdpqf7fcm-stack-AWSEBCloudwatchAlarmHigh-4AXGHSGTZFIQ

2022-07-29 20:51:13    INFO    Deleted CloudWatch alarm named: awseb-e-vzdpqf7fcm-stack-AWSEBCloudwatchAlarmLow-9TPSITYGQX56
2022-07-29 20:51:13    INFO    Deleted Auto Scaling group policy named: arn:aws:autoscaling:us-west-2:349327579537:scalingPolicy:62397ed2-02a4-4dcd-bc57-d352ad65b780:autoScalingGroupName/awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingGroup-O5BEPBX3UO1Z:policyName/awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingScaleUpPolicy-QrGFM41rCbk4
2022-07-29 20:51:13    INFO    Deleted Auto Scaling group policy named: arn:aws:autoscaling:us-west-2:349327579537:scalingPolicy:76cf55ec-df94-4d28-9d6b-b7f488b0471f:autoScalingGroupName/awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingGroup-O5BEPBX3UO1Z:policyName/awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingScaleDownPolicy-VpH1cGs0NFvC
2022-07-29 20:51:13    INFO    Waiting for EC2 instances to terminate. This may take a few minutes.
2022-07-29 20:52:44    INFO    Deleted Auto Scaling group named: awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingGroup-O5BEPBX3UO1Z
2022-07-29 20:52:44    INFO    Deleted load balancer named: awseb-e-v-AWSEBLoa-11E3Z8HYHWI7M
2022-07-29 20:52:44    INFO    Deleted Auto Scaling launch configuration named: awseb-e-vzdpqf7fcm-stack-AWSEBAutoScalingLaunchConfiguration-gdwq18IM1imP
2022-07-29 20:52:44    INFO    Deleted security group named: awseb-e-vzdpqf7fcm-stack-AWSEBSecurityGroup-14EBI3MI8MEZG
2022-07-29 20:53:30    INFO    Deleted security group named: sg-0d4e57ecfe78dd7f3
2022-07-29 20:53:33    INFO    Deleting SNS topic for environment aws-ebeanstalk-py-django-01-env.
2022-07-29 20:53:34    INFO    terminateEnvironment completed successfully.
```

- Delete the following resources manually from the AWS console:
  - Elastic Beanstalk application: `aws-ebeanstalk-py-django-01`
  - Amazon S3 - elasticbeanstalk-*** > `aws-ebeanstalk-py-django-01` folder
    - NOTE: need to "Empty" the bucket first, then delete the "Bucket Policy", 
      before deleting the bucket.
 
- [Optional] investigate whether the followings can be deleted: 
  - Amazon S3 - elasticbeanstalk-***; NOTE: need to "Empty" the bucket first, then delete the "Bucket Policy", before deleting the bucket. 

## References

- REF-1: [AWS - Install the EB CLI in a virtual environment](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-virtualenv.html)

- REF-2: [AWS - Install Python, pip, and the EB CLI on Linux](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-linux.html)
