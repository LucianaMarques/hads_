# HADS

This repository contains all files related to the Hibernation-Aware Dynamic Scheduler (HADS)

## Installation Guide

This installation guide describes a verified process for Linux environment (Ubuntu 20.04 LTS) and MACOS. All instructions reported here are based on this system. Number versions are provided as an indication of the versions that were tested and used in this project. 


You must have Docker installed and running as well as Python 3.

- Docker (v20.10.17)
- Python 3 (v3.9)

:warning: Currently HADS only supports the services of AWS. All the instalation steps related to the cloud is based on this provider.


### Installing the controller

The controller is responsabel for all the management of the virtual machines in the cloud: it request machines, deploy the tasks and terminate the environment. Before install the control, you need to generate a Identity and Access Management (IAM) 

1. Execute the script `install.sh`  in `install/control/`. First, update `SOURCE_PATH` variable inside it with the full path to the `hads_` project;

2. If all goes well, test if client is working in the `hads/` source repo: `python clienty.py --help`.

### Environment variables

To execute the controller, it is necessary to define the following environment variables:

* The full path to the controler
    * `HADS_PATH=/home/hads_ `
* The path and the file name where the setup file is
    * `SETUP_PATH=/home/hads_/`
    * `SETUP_FILE=setup.cfg`
* The user name and the password used in the database
    * `POSTGRES_USER=<postgres_user>`
    * `POSTGRES_PASS='<postgres_pwd>'`



### AWS setup

After setup controller, it is necessary to define the permissions in AWS and create an AMI used to execute the tasks.


#### AWS permissions and setup

It is necessary to have the following AWS permissions and setup:

1. AWS CLI
2. AWS Access Key and Secret id (usually stored in ~/.aws/credentials
3. EC2 creation and deletion permissions
4. pricing:GetProducts

#### Secury group

It is also necessary to create a security group that will allow HADS to access the VMs via ssh

To create the security group, go to EC2, Network & Security and access the Security Groups.

Create a security group by defining its name and creating an inbound rule that allows all TCP access.

The name of the created security group needs to be included in the `setup.cfg` file in the tag:

```
[ec2]
security_group = <security group name>
```

### SSH key

To access the VMs, it is necessary to create an ssh key in EC2. To create the key, go to EC2, key pairs and select the button Create key pair. Defines the name of the key and selects "create the key pair". After that, you will download a file .pem to your local machine (where the controller was installed), and it is necessary to give permission 400 to this file.

```bash
chmod 400 <file.pem>
```

The name of the created key needs to be included in the `setup.cgf` file in the tag:

```
[ec2]
key_name = hads-key
```

Moreover, you also need to give the full path to the .pem file in the `setup.cfg` in the tag:

```
[communication]
key_path = <full path to the .pem file>
key_file = <file.pem>
```


### S3 Bucket

By default, HADS uses an S3 bucket to keep all files related to the programs and all output files. Therefore, it is necessary to create a bucket in S3.

In S3, select the create bucket buttom. Then, defines the bucket name and create the bucket (the default bucket setup is enough for HADS).

The name of the created bucket needs to be included in the `setup.cfg` file in the tag:

```
[ec2]
bucket_name = <s3 bucket>
```


### Creating an AMI

The Amazon Machine Images (AMI) is an image provided by AWS that contains all information required to launch an instance. To execute applications in AWS, HADS launched a pre-defined AMI that contains all dependencies necessary to execute the applications.

Currently, all applications are executed in a Docker container. Therefore, we need to set up an AMI with Docker and create a container able to execute the application.

Besides, the communication between the VM and the S3 bucket is made using the s3fs-fuse (https://github.com/s3fs-fuse/s3fs-fuse).

All the steps described here were made using an AMI running ubuntu 18.04. For all the next steps, you can use a t2.micro instance (usually free):

To create an AMI in AWS:

1. Launch a VM with ubuntu 18.04. 
    * Selects the previously created ssh-key and security group
    * Define storage with enough capacity to keep all applications files (we cannot change it after the AMI creations): 30 GB should be enough for most of the applications.

2. In the VM, execute the script `prepare_ami.sh` available in `install/ami/`

:warning: the script `prepare_ami.sh` requires all files in the folder `install/ami/`


The script will ask for the IAM access key and secret key created in the first step of this installation guide. That information can be found in the `~/.aws/credentials` file.


After the script is finished, it is necessary to restart the VM

```bash
sudo reboot -h 0
```

Now, you need to create a docker image that will be used to execute the application. As an example, for the next steps, we will consider a synthetic application available in `bin/example/`.

To prepare a docker image able to execute that application,  use the DockerFile available in `install/ami/synthetic`. In that folder, you can create an image called "synthetic" executing the following command:

```bash
docker build -t synthetic .
```

The name of the created docker image needs to be included in the `setup.cfg` file in the tag:

```
[docker]
docker_image = synthetic
```


Now, we can finally create the AMI by saving the VM state:

1. In the EC2 console, select the running VM.
2. In the button "actions" "Image and templates", select create image
3. Define a name for the image and create the image

That process takes several minutes! You can check the status of the image in "images", and "AMIs". The image is ready to be used only when the status changes from pending to available.

:warning: Do not terminate the VM while the image is not completely created.

Note that, the created image has an AMI ID (available even before the end of the image creation). The AMI ID of the image needs to be included in the `setup.cfg` file in the tag:

[ec2]
image_id = 	<ami-id>

:warining: Do not forget to terminate the VM when the image is finally created.


## HANDS ON: Running the synthetic application

After all the installation steps, we can check if HADS can execute applications in the cloud by testing the execution of the synthetic application available in `bin/example/`






## Published Papers
-   Teylo, L.; Arantes, L.; Sens, P.; Drummond, L. A Hibernation Aware Dynamic Scheduler for Cloud Environments. _15th International Workshop on Scheduling and Resource Management for Parallel and Distributed Systems_, ICPP, 2019.
-   Teylo, L.; Arantes, L.; Sens, P; Drummond, L. A dynamic task scheduler tolerant to multiple hibernations in cloud environments._Cluster Computing_, v. 4, p. 1-23, 2020  (Available Online).
-   Teylo, L.; Brum, R.; Arantes, L.; Sens, P.; Drummond, L. Developing Checkpointing and Recovery Procedures with the Storage Services of Amazon Web Services.  _16th International Workshop on Scheduling and Resource Management for Parallel and Distributed Systems_, ICPP, 2020.
-   Teylo, L.; Nunes, A. L.; Melo C. M. A., A.; Boeres, C.;  Drummond, L.; Martins, N. Comparing sars-cov-2 sequences using a commercial cloud with a spot instance based dynamic scheduler.  _21th IEEE/ACM International Symposium on Cluster, Cloud and Grid Computing_, 2021.


