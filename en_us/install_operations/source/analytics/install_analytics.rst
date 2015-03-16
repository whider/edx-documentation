.. include:: ../links.rst

.. _Installing edX Insights:

#######################
Installing edX Insights
#######################

This chapter is intended for those who are interested in running `edX Insights`_
and its dependencies in a production environment. Work to prepare complete
installation procedures for edX Insights is in progress. Introductory material
is available now.

.. contents:: Chapter Contents:


********
Overview
********

Course teams use edX Insights to access to data gathered from active courses.
Course teams use edX Insights to display charts, summary statistics, and data
tables.

The Learning Management System (LMS) gathers data about student activity. This
data is aggregated by the edX Analytics Pipeline. The aggregated data is
exposed by the `edX Analytics Data API`_. EdX Insights reads the data from the
edX Analytics Data API and presents the data to course team members.

.. _edX Analytics Data API: http://edx.readthedocs.org/projects/edx-data-analytics-api/en/latest/index.html

============
Architecture
============

.. image:: /Images/Analytics_Pipeline.png
 :alt: Image showing the relationships between various components of the edX
       analytics data pipeline.

==========
Components
==========

LMS
***

The LMS records student actions in tracking log files. The standard
``logrotate`` utility periodically compresses and copies these files into a
filesystem that can be read by the edX Analytics Pipeline. The LMS also
captures a lot of information in a MySQL database. The edX Analytics Pipeline
connects directly to this database to extract information about students.

edX Analytics Pipeline
**********************

The edX Analytics Pipeline reads the MySQL database used by the LMS as well as
the tracking log files produced by the LMS. The data is processed and the
resulting summary data is published to the result store. The result store is a
MySQL database.

Requirements:

* `Hadoop <http://hadoop.apache.org/>`_ version 1.0.3 or higher
* `Hive <https://hive.apache.org/>`_ version 0.11.0.2 or higher
* `Sqoop <http://sqoop.apache.org/>`_ version 1.4.5
* Python 2.7
* Either Debian version 6.0 or higher, or Ubuntu version 12.04 or higher.
* A MySQL server version 5.6 or higher

Scheduler
*********

The Scheduler schedules the execution of data computation tasks. Data
computation tasks are run by the edX Analytics Pipeline. Data computation tasks
are used to update parts of the result store.

edX Analytics Data API
**********************

The edX Analytics Data API provides an HTTP interface for accessing data in the
result store. Typically, the data in the result store is updated periodically by
the edX Analytics Pipeline.

Requirements:

* Python 2.7

edX Insights
************

EdX Insights uses the edX Analytics Data API to present data to users. Users
access the data using a supported web browser. EdX Insights communicates
directly with the LMS to authenticate users, authorize
users and read course structure information.

Requirements:

* Python 2.7

*************************************
What You Should Know Before You Start
*************************************

You must understand the following concepts to install edX Insights and deploy
the edX Analytics Pipeline:

* Understand basic terminal usage.
* Understand how the LMS has been deployed and configured.
* Understand basic computer network terminology.
* Understand the YAML file format.
* Understand Amazon Web Services terminology.


************************
Planning Your Deployment
************************

All edX Analytics services are designed to be relocatable. This means that they
do not require a particular configuration of virtual servers. You are free to
choose how the services should be distributed among the resources you have
available.

======
Hadoop
======

Most of the computation performed by the edX Analytics Pipeline is implemented
as Map Reduce jobs that currently must be executed by a Hadoop cluster. You can
scale your Hadoop cluster based on your current and projected data sizes. Hadoop
clusters can be scaled vertically and horizontally as your data grows. For very
small installations of Open edX, a single virtual server should be sufficiently
powerful to process your data.

Amazon's `Elastic MapReduce`_ service offers pre-configured Hadoop clusters. If you
are able to use Amazon Web Services, use of this service is recommended. Proper
installation and configuration of Hadoop can be time consuming.

Additionally, vendors such as Cloudera and MapR offer simplified Hadoop
administration experiences.

Hadoop is a distributed system that consists of several different services. It
is worth noting that they are Java services and require a non-trivial amount of
memory to run. The high memory requirement may prevent you from running all
services on the same virtual server if it does not have enough memory available.

.. _Elastic MapReduce: http://aws.amazon.com/elasticmapreduce/

================
edX Applications
================

The edX Analytics Data API responds to a small number of requests every time a
page is loaded in edX Insights. Small installations can probably host both
services on the same virtual server. Larger installations will want to consider
hosting them on more than one virtual server. A load balancer is recommended for
each service that requires more than one virtual server.

============
Result Store
============

The results of computations performed by the edX Analytics Pipeline are stored
in a MySQL database. Even small installations should use a different MySQL
server than the one used by the LMS. The query patterns of the edX Analytics API
are more I/O intensive than usual. Placing both databases on the same server may
degrade performance of the Learning Management System.

=========
Scheduler
=========

Scheduling executions of the edX Analytics Pipeline can be accomplished in many
different ways. Any tool that can periodically execute shell commands should
work. The simplest tool that can perform this task is `cron`_. `Jenkins`_ is
also a good candidate.

.. _cron: http://en.wikipedia.org/wiki/Cron
.. _Jenkins: http://jenkins-ci.org/

*******************
Example Deployments
*******************

===================================
Small Scale Using Elastic MapReduce
===================================

A small deployment might consist of a single master node and a single core node.
The Scheduler is deployed to the master node and periodically executes the edX
Analytics Data Pipeline on this server. Additionally, the edX Analytics API, edX
Insights and result store are deployed to the master node. These services run
continuously.

===================================
Large Scale Using Elastic MapReduce
===================================

A large scale deployment consists of a single master node, several core nodes,
and many task nodes deployed into a public subnet of a Virtual Private Cloud.

.. image:: /Images/Analytics_AWS_Deployment.png
 :alt: Image showing all of the AWS components needed to run a large scale
       deployment of the edX analytics data pipeline.

The edX Analytics API and edX Insights are each deployed into an auto-scaling
group behind an Elastic Load Balancer which terminates SSL connections and
distributes the load among the application servers. The application servers are
deployed into a private subnet of the Virtual Private Cloud. A single virtual
server is deployed into a private subnet to host the Scheduler. The Relational
Database Service is used to deploy a MySQL server into a private subnet. The
MySQL database will be used as the result store.

===========================================
Large Scale Without Using Elastic MapReduce
===========================================

A large deployment that does not use Elastic MapReduce requires additional
configuration steps to establish a properly configured environment. A Hadoop
cluster is deployed. The master node is considered the node that is running the
Job Tracker service for Hadoop 1.X deployments or the Resource Manager service
for Hadoop 2.X deployments. Hive and Sqoop are deployed to the master node.
Several servers are deployed outside of the Hadoop cluster that host the
remainder of the infrastructure. The edX Analytics API and edX Insights services
are each deployed to at least one server. The Scheduler is deployed to another
server. A MySQL database is deployed to a server that is configured to host a
relational database.


******************************
Using Elastic MapReduce on AWS
******************************

1. A VPC (Virtual Private Cloud) is created with at least one public subnet. A
   limitation of EMR (Elastic MapReduce) running in a VPC is that clusters must
   be deployed into public subnets of VPCs. An existing VPC may be used if
   one is already available for use.

    a) It is recommended, but not necessary, to also have at least one private
       subnet in addition to the required public subnet. See the AWS
       documentation for an `example configuration
       <http://docs.aws.amazon.com/AmazonVPC/latest/User
       Guide/VPC_Scenario2.html>`_.
    #) An example configuration that only includes a single public subnet can be
       found in `this document <http://docs.aws.amazon.com/AmazonVPC/latest/User
       Guide/VPC_Scenario1.html>`_ published by AWS.
    #) It can also be advantageous to deploy several public subnets in different
       availability zones in order to take advantage of price fluctuations in
       the `spot pricing market <http://aws.amazon.com/ec2/purchasing-options
       /spot-instances/>`_.
    #) Consider that the clusters deployed using EMR will need network
       connectivity to a read replica of the LMS database. Using the same VPC as
       the LMS is recommended if possible.
    #) It is possible to run all of these services outside of a VPC but not
       recommended.

#. An IAM role is created for use by the EMR cluster. All EC2 nodes in the
   cluster will be assigned this role. Consider copying the contents of the
   `default role
   <http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/emr-iam-
   roles-defaultroles.html#emr-iam-roles-defaultec2>`_ used by AWS.

#. An IAM user with administrative privileges is available for use.

#. A secure ssh key is generated. This will be used to access all AWS resources.

    .. code-block:: bash

        ssh-keygen -t rsa -C "your_email@example.com"

#. The public key from the SSH key pair is uploaded to AWS and assigned a
   keypair name.

#. Ensure at least one EC2 instance is available to host the various services.
   Depending on the scale of the deployment, more instances may be necessary.
   The instances should be deployed using the secure keypair name.

#. A MySQL 5.6 RDS instance is deployed. This is the result store.

    a) Ensure there is connectivity between the EC2 instance that is hosting the
       edX Analytics API and this RDS instance. Also, ensure that instances
       deployed into the public subnet of the VPC will be able to connect to
       this RDS instance.
    #) At least two users are created on this instance. One is called "pipeline"
       and has permission to create databases, tables, indexes and issue
       arbitrary DML (Data Manipulation Language) commands. The other, called
       "api", has read-only access. Passwords are configured for both of these
       users.
    #) This instance is configured to use the "utf8_bin" collation by default
       for columns in all databases and tables.

#. An SSH connection is established to the EC2 instance within the VPC that will
   run the scheduler service. All further commands described below should be
   issued from a shell running on that instance.

#. The sources files from `edx-analytics-configuration <https://github.com/edx
   /edx-analytics-configuration>`_ are checked out.

    .. code-block:: bash

        git clone https://github.com/edx/edx-analytics-configuration.git

#. The shell is configured to use the AWS credentials of the administrative AWS
   user.

    .. code-block:: bash

        export AWS_ACCESS_KEY_ID=<access key ID goes here>
        export AWS_SECRET_ACCESS_KEY=<secret access key goes here>

#. The `AWS command line utility <http://aws.amazon.com/cli/`_ is installed.

    .. code-block:: bash

        pip install awscli

#. An S3 bucket is created to hold all Hadoop logs from the EMR cluster.

    .. code-block:: bash

        aws s3 mb s3://<your logging bucket name here>


#. An S3 bucket is created to hold secure configuration files and initialization
   scripts.

    .. code-block:: bash

        aws s3 mb s3://<your configuration bucket name here>

#. The `MySQL connector library
   <http://dev.mysql.com/downloads/connector/j/5.1.html>`_ is downloaded from
   Oracle. Once the download is completed it is uploaded to S3.

    .. code-block:: bash

        aws s3 cp /tmp/mysql-connector-java-5.1.*.tar.gz s3://<your configuration bucket name here>/

#. Since the MySQL connector library version has likely changed from when the
   script was written, the ``edx-analytics-configuration/batch/bootstrap
   /install-sqoop`` script may need to be modified to point to the correct
   version of the library in the S3 bucket. That change must be made before
   continuing.

#. The contents of the directory ``edx-analytics-
   configuration/batch/bootstrap/`` are uploaded into the ``openedx-analytics-
   configuration`` bucket.

    .. code-block:: bash

        aws s3 sync edx-analytics-configuration/batch/bootstrap/ s3://<your configuration bucket name here>/

#. A cluster configuration file is created to specify the parameters for the
   cluster that will soon be created. All of these parameters are reviewed and
   changed appropriately to specify the desired configuration. This file is
   saved to a temporary location such as ``/tmp/cluster.yml``.

    .. code-block:: yaml

        {
            name: <your cluster name here>,
            keypair_name: <your keypair name here>,
            vpc_subnet_id: <your VPC public subnet ID here>,
            log_uri: "s3://<your logging bucket name here>",
            instance_groups: {
                master: {
                    num_instances: 1,
                    type: m1.medium,
                    market: ON_DEMAND,
                },
                core: {
                    num_instances: 2,
                    type: m1.medium,
                    market: SPOT,
                    bidprice: 0.03
                },
                task: {
                    num_instances: 1,
                    type: m1.medium,
                    market: SPOT,
                    bidprice: 0.03
                }
            },
            bootstrap_actions: {
                security: {
                    path: "s3://<your configuration bucket name here>/security.sh"
                },
                jobtrackerconfig: {
                    path: "s3://elasticmapreduce/bootstrap-actions/configure-hadoop",
                    args: [
                        -m, "mapred.jobtracker.completeuserjobs.maximum=5",
                        -m, "mapred.job.tracker.retiredjobs.cache.size=50",
                        -m, "mapred.job.shuffle.input.buffer.percent=0.20"
                    ]
                }
            },
            steps: [
                { type: hive_install },
                {
                    type: script,
                    name: Install Sqoop,
                    step_args: [
                        "s3://<your configuration bucket name here>/install-sqoop",
                        "s3://<your configuration bucket name here>"
                    ]
                }
            ],
            user_info: []
        }

#. An EMR cluster is deployed.

    .. code-block:: bash

        EXTRA_VARS="@/tmp/cluster.yml" make provision.emr

    Example Output::

        pip install -q -r requirements.txt
        ansible-playbook --connection local -i 'localhost,' batch/provision.yml -e "$EXTRA_VARS"

        PLAY [Provision cluster] ****************************************************** 

        TASK: [provision EMR cluster] ************************************************* 
        changed: [localhost]

        TASK: [add master to group] *************************************************** 
        ok: [localhost]

        TASK: [display master IP address] ********************************************* 
        ok: [localhost] => {
            "msg": "10.0.1.236"
        }

        TASK: [display job flow ID] *************************************************** 
        ok: [localhost] => {
            "msg": "j-29UUJVM8P1NPY"
        }

        PLAY [Configure SSH access to cluster] **************************************** 

        TASK: [user | debug var=user_info] ******************************************** 
        ok: [10.0.1.236] => {
            "item": "", 
            "user_info": []
        }

        TASK: [user | create the edxadmin group] ************************************** 
        changed: [10.0.1.236]

        TASK: [user | ensure sudoers.d is read] *************************************** 
        changed: [10.0.1.236]

        TASK: [user | grant full sudo access to the edxadmin group] ******************* 
        changed: [10.0.1.236]

        TASK: [user | create the users] *********************************************** 
        skipping: [10.0.1.236]

        TASK: [user | create .ssh directory] ****************************************** 
        skipping: [10.0.1.236]

        TASK: [user | assign admin role to admin users] ******************************* 
        skipping: [10.0.1.236]

        TASK: [user | copy github key[s] to .ssh/authorized_keys2] ******************** 
        skipping: [10.0.1.236]

        TASK: [user | copy additional authorized keys] ******************************** 
        skipping: [10.0.1.236]

        TASK: [user | create bashrc file for normal users] **************************** 
        skipping: [10.0.1.236]

        TASK: [user | create .profile for all users] ********************************** 
        skipping: [10.0.1.236]

        TASK: [user | modify shell for restricted users] ****************************** 
        skipping: [10.0.1.236]

        TASK: [user | create bashrc file for restricted users] ************************ 
        skipping: [10.0.1.236]

        TASK: [user | create sudoers file from template] ****************************** 
        changed: [10.0.1.236]

        TASK: [user | change home directory ownership to root for restricted users] *** 
        skipping: [10.0.1.236]

        TASK: [user | create ~/bin directory] ***************************************** 
        skipping: [10.0.1.236]

        TASK: [user | create allowed command links] *********************************** 
        skipping: [10.0.1.236]

        PLAY RECAP ******************************************************************** 
        10.0.1.236                 : ok=0    changed=4    unreachable=0    failed=0   
        localhost                  : ok=4    changed=1    unreachable=0    failed=0

