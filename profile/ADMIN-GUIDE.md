![Frontend](img/wod-frontend.png "Frontend")

Leveraging the JupyterHub technology, the HPE Developer Community team, at the origin of the project, created a set of hands-on technology training workshops where anyone could connect to our service and benefit from notebook-based workshops at any time, and from anywhere. Read our [User's Guide](USER-GUIDE.md) to see it in action.

Now, let me explain how we made this possible…

# Where it all comes from

We built up our first JupyterHub server for the virtual HPE Technology & Solutions Summit (TSS) that took place in March 2020 and used it to deliver the workshops we would normally give in person in a virtual way. We hosted just a few Jupyter notebooks-based workshops and found that they scored really well in feedback we received from the students. All notebooks were delivered through a single instance of a JupyterHub server.

During the event, we ran only a single workshop at a time. We first set up a range of users (students) to match a given workshop. It required a lot of manual setup, a few scripts and a single Ansible playbook to handle the copy of the workshop content assigned to the desired range of students, with a few variable substitutions like student IDs, passwords, and some API endpoints when relevant. The student assignment was done manually, at the beginning of every workshop. For instance, Fred was student1, Didier was student2, and so on… which was quite cumbersome. When you only have a range of 25 students, one or two people handling the back end is sufficient. But if 40 people show up, then it becomes tricky.

This initial work lead us the following results collected in our dashboard presenting:
-    The number of Active Workshops
-    The active workshops by type
-    The total number of registrations from November 1st 2020 till March 30th 2021
-    The total number of Customer (student) registrations
-    The total workshops split
![Hackshack Dashboard](img/hackshack-dashboard.png "Hackshack Dashboard")

As you can see, moving from a heavily manually oriented approach to now a fully automated one helped increase our numbers. By the end of 2022, more than 4000 had registered for our workshops.

Of course, as automation minded people, we could not stay like that for long, so we decided to build a framework to help us deliver more Workshops on Demand (aka WoD) and began our 5 years journey of developing it till version 1.0.0 :-)

# Architecture and technologies considerations

We soon realized that for the project to be usable, we would need to improve various aspects. Our needs were the following:
- to provide a registration portal to allow workshop exposure and automated user registration.
- to automate fully all the tasks on the platform (deployment of the notebook, once chosen on that portal, cleanup, security, ...)
- to keep track of the association made between users and students, the workshop chosen,
- to manage workshops (capacity, reset specificities, ...)
- to add easily more content to the platform, developed by contributors
- to manage a private set of data (content, parameters, scripts) in addition to the public one provided
- to support mulitple locations for redundancy, technology specificities
- to operate multiple platforms (dev, staging, production, ...)
- to operate as well on VMs or physical servers
- to deploy the platform from zero to ready to operate fully automated

As you can see, we have a few environments to deploy and manage. The various aspects were coded during 5 years to reach the desired state expressed upper. Without automation, it would have been too much work and prone to error. That was a key aspect of our approach.

We quickly chose the various tools that will help us manage this efficiently:
- Linux for the OS ([Ubuntu LTS](https://ubuntu.com/download/server) being the primary distribution, CentOS/[Rocky Linux](https://rockylinux.org/download) the alternative for backend)
- [Ansible](https://en.wikipedia.org/wiki/Ansible_(software)) for automation and conformity management
- [YAML](https://en.wikipedia.org/wiki/YAML) for configuration files (wod.yml, ansible playbooks and variables)
- [Git](https://en.wikipedia.org/wiki/Git) as a configuration management system storing all our files and [GitHub](https://github.com/Workshops-on-Demand) for publication
- [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) to store permanently some information around users, workshops, students, ...
- [Javascript](https://en.wikipedia.org/wiki/JavaScript), [NodeJS](https://en.wikipedia.org/wiki/Node.js), [React](https://en.wikipedia.org/wiki/React_(software)), [Grommet](https://v2.grommet.io/) for the frontend and API servers
- [REST](https://en.wikipedia.org/wiki/REST) API (The one from JupyterHub, and one we created for our project)
- [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) API using [procmail](https://en.wikipedia.org/wiki/Procmail) for our incoming communication with the backend

From an architecture perspective, we rapidly agreed that we would need multiple machines/services:
![Multiple servers/services](img/wod-infra-open-source.png "Multiple servers/services")
- A frontend server to manage the workshop list and allow user registration called wod-frontend
- An API server coupled with the Database to orchestrate the platform called wod-api-db
- A backend server on which will run the JupyterHub service and where the Notebooks will be deployed called wod-backend
- Appliances servers running the technology at work in the Notebook (optional)

For our automation, we have Ansible playbooks that would perform:
- Server configuration (based on an existing deployed Operating System with just git installed) including:
    - Repository Update
    - Apps Installation
    - System Performance Tuning including kernels setup & configuration
    and depending on the server:
    - on wod-backend:
        - JupyterHub application installation and configuration
        - Linux students creation
        - JupyterHub users creation
        - A complete JupyterHub server preinstalled and configured with some additional Jupyterhub kernels
        - A postfix server used for the procmail API
    - on wod-frontend:
         - Javascript setup with npm
         - portal installation and launch
    - on wod-api-db:
         - Javascript setup with npm
         - PostgreSQL install and setup with sequelize
         - REST API server installation and launch
- Server conformity management (run after the previous playbook and on demand and nightly to ensure that the configuration is consistent and up to date):
    - System Update
    - Security Setup
    - Services Conformity and check (apps, users)
    - Templating of configration and data files
    - A fail2ban service  to limit abuses
    - An Admin user to manage everything
- Notebooks deployment on the JupyterHub server:
    - Templating the Notebook and data files
    - Security setup


The Workshops-on-Demand GitHub repositories can be found [here](https://github.com/Workshops-on-Demand/).
![GitHub WoD repositories](img/wod-repositories-2025.png "GitHub WoD repositories")

We created as many Git repositories as needed, for the infrastructure management:
![WoD repositories](img/wod-repositories.png "WoD repositories")
- [wod-frontend](https://github.com/Workshops-on-Demand/wod-frontend) for the WoD portal based on NGINX and NodeJS technologies, it provides the participtants' Registration Portal used to enable booking of the workshops.
- [wod-api-db](https://github.com/Workshops-on-Demand/wod-api-db) for the REST Open API 3.0 based API and a PostgreSQL DB hosting the different status of participants, workshops, and students.
- [wod-backend](https://github.com/Workshops-on-Demand/wod-backend) for the JupyterHub and Postfix mail server.
- [wod-notebooks](https://github.com/Workshops-on-Demand/wod-notebooks) for the public Jupyter based Notebooks provided. You can test them live at <https://hackshack.hpedev.io/workshops>
- [wod-private](https://github.com/Workshops-on-Demand/wod-private) as a template for using a WoD private setup alongside the public one, including a cutomization layer on top of the public standard wod-backend / WoD Notebooks content. Do not put any confidential data here as this is a public repository!
- [wod-install](https://github.com/Workshops-on-Demand/wod-install) for the installation of the WoD infrastructure, allowing you to install either wod-backend, wod-api-db or wod-frontend servers using a single line of command.
- [wod-doc](https://github.com/Workshops-on-Demand/.github) for the project documentation (you're on it !)

**Note:** *There are 7 repositories available for now.*

# How it works

Let us first show you how the overall process works for our Workshops-on-Demand:

![Workshops-on-Demand principles](img/wod-principles.png "Workshops-on-Demand principles")

If you’re looking for a live explanation of this automation, you can review [the following session](https://www.youtube.com/watch?v=D6Ss3T2p008&t=515s) delivered at Linuxconf in Australia in January 2021. If you prefer reading, consider our [User's Guide](USER-GUIDE.md). Here we'll go into more technical details on how this performed:

**Step 1:** The customer registers for a workshop online at our portal. When clicking the register button for the selected workshop and after agreeing to the terms and conditions, the wod-frontend triggers the first REST API calls to the wod-api-db server:
-    Register the student in the Customers Database.
-    Send a welcome email to the student.
-    Assign a student ID to him/her according to the student range allocated to the given workshop.

**Step 2:** The wod-api-db server then orders (through a mail API call managed by procmail) the wod-backend to:
-    Generate a random password for the selected student
-    Deploy the selected workshop

**Step 3:** The wod-backend server calls back the wod-api-db server using its REST API to:
-    Provide back the new student password
-    Make the student’s database record active
-    Decrement Workshop capacity

**Step 4:** The wod-api-db server sends:
-    The credentials email to allow the student to connect to the workshop

This infrastructure automation relies mainly on a few bash scripts and Ansible playbooks to:
-    Generate random passwords
-    Update LDAP passwords accordingly when required
-    Deploy the given workshop to the proper student home directory through an Ansible playbook
-    Update the notebook content through Ansible variables substitutions
     -    Student ID
     -    Student password
     -    API Endpoints definition
-    Update student permissions
-    Perform all necessary create or reset actions linked to a given workshop outside of the notebook customization

The wod-api-db server actually sends an email to the wod-backend with a dedicated format (an API!) that is then parsed by the Procmail process to retrieve the necessary information to perform the different tasks. We use 4 verbs: CREATE (to setup to the student environment as described upper), DELETE (to remove the user from tables), RESET (to clean up the student content and reset the back-end infrastructure when needed) and PURGE (globally DELETE+RESET aka full cleanup)

At the end of this automated process, the wod-backend makes a series of API calls to the wod-api-db server to send back the required information, like a new password, workshop status, etc.

# A bit of background

This solution allows us to deliver free hands-on workshops. It has been used during events (HPE Discover, HPE TSS, as well as Open Source Summit, KubeCon), and during training sessions (HPE trainings, RMLL, Campus Numérique, ...) to promote API/automation-driven solutions along with some 101-level coding courses.

If you are interested in creating your own training architecture to deliver workshops, this project is definitely for you. It would allow you to create, manage content and deliver it in an automated and very efficient way. Once ready, the infrastructure can deliver workshops in matter of minutes!

Trying to build a similar architecture on your own is obviously possible, but we integrated so much automation around the different components like the JupyterHub server deployment along with multiple pre installed kernels, user management and much more. When leveraging our project, one can actually get a proper public-based environment in 2 hours.

## Why would we consider open sourcing our Workshops-on-Demand?

Firstly, if you read carefully the messaging on our [homepage](https://developer.hpe.com/) , you will find words like sharing and collaborating. This is part of the HPE Developer team's DNA.

Secondly, the project is based on open source technologies like Jupyter and Ansible. It felt natural that the work we did leveraging these should also benefit the open source community.

We have, actually, shared the fundamentals of the project thoughout the HPE Developer Community, and to a wider extent, the Open Source Community  through different internal and external events. And the feedback has always been positive. Some people found the project very appealing. Originally, and long before even thinking of open sourcing the project, when we really started the project development, people were mainly interested in the content and not necessarily in the infrastructure. The students wanted to be able to reuse some of the notebooks. And in a few cases, they also asked for details about the infrastructure itself, asking about the notebooks delivery mechanism and other subjects like the [procmail API](https://www.youtube.com/watch?v=zZm6ObQATDI).

In 2022, we were contacted by an HPE colleague who was willing to replicate our setup in order to deliver notebooks to its AI/ML engineers. His aim was to provide a simple, central point from which he could deliver Jupyter Notebooks, that would later be published on the Workshops-on-Demand infrastructure wod-frontend portal, allowing content to be reused and shared amongst engineers. While, over time, we had worked  a lot on automating content delivery and some parts of the infrastructure setup, we needed now to rework and package the overall solution to make it completely open source and reusable by others.

As a result of our work on that project, over the course of 2022 we started to open source the Workshops-on-Demand program. As a project developed within the confines of Hewlett Packard Enterprise (HPE), we had a number of technical, branding, and legal hurdles we needed to overcome in order to achieve this.

### Legal side of things

From a legal standpoint, we needed to go through the HPE OSRB (Open Source Review Board) to present the project that we wanted to open source. We were asked to follow a process that consisted of four steps:

![HPE OSRB Process](img/wod-osrb.png "HPE OSRB Process")

As the project did not contain any HPE proprietary software, as it is based on open source technologies like Ansible and Jupyter, the process was quite straightforward. Besides, HPE did not want to exploit commercially the generated intellectual property.  We explained to the OSRB that the new architecture of the solution would allow the administrator of the project to separate public content from private content, with private content being a proprietary technology.

This had a huge influence on the future architecture of the project that originally did not allow it. In our case, for instance, any workshop related to an HPE technology like  HPE Ezmeral, would fall into the private part of the project, and therefore, would not appear on the public github repository that we had to create for the overall project distribution.

### Technical side of things

From a technical standpoint, as mentioned above, we had to make sure to separate the public only content from any possible private content. We started by sorting the different workshops (public vs private based). We also had to sort the related scripts that come along with the workshops. Going through this process, we found out that some of the global scripts had to be reworked as well to support any future split of public and private content. Similarly, we had to address any brand specifics, parameterizing instead of hardcoding variables as it was in the first version.

This took us a few months and we are now ready to share with you the result of this work. We will now focus on the architecture side of the Workshops-on-Demand project.

# The WoD Architecture

 The Workshops-on-Demand concept is fairly simple. The following picture gives you a general idea of how the process works.

![Workshops-on-Demand Concepts High Level View](img/wod-howto.png "Workshops-on-Demand Concepts High Level View")

Now that you understand the basic principle, let's look at the details. The image below shows what happens at each stage from a protocol standpoint.

![Workshops-on-Demand Architecture and Protocols Diagram](img/wod-protocols.png "Workshops-on-Demand Architecture and Protocols Diagram")

## The Register Phase

**1.** Participants start by browsing a wod-frontend web server that presents the catalog of available workshops. They then select one and register for it by entering their email address, first and last names, as well as their company name. Finally, they accept the terms and conditions and hit the register button.

As the register button is clicked, the wod-frontend server performs a series of actions.

**2.** Assigns a student (i.e student401) from the dedicated workshop range to the participant. Every workshop has a dedicated range of students assigned to it.

Here is a screenshot of the workshop table present in the wod-api-db database server showing API 101 workshops details.

![Workshops Table from wod-api-db server](img/wod-db-workshop-table.png "Workshops Table from wod-api-db server")

* Frederic Passeron gets assigned a studentid "student397" for workshop "API101".

  Here are the details of the participant info when registered to a given workshop.

![Customers Table from wod-api-db server](img/wod-db-customer-table.png "Customers Table from wod-api-db server")

**3.** An initial email is sent to participants from the wod-api-db server welcoming them to the workshop and informing them that the deployment is ongoing and that a second email will arrive shortly providing the necessary information required to log onto the workshop environment.

**4.** At the same time, the wod-api-db server sends the necessary orders through a procmail API call to the wod-backend server. The mail sent to the wod-backend server contains the following details:

* Action Type ( CREATE, CLEANUP, RESET, PURGE)
* Workshop ID
* Student ID

**5.** The wod-backend server recieves the order and processes it by parsing the email recieved using the procmail API. The procmail API automates the management of the workshops.

Like any API, it uses verbs to perform tasks.

* CREATE to deploy a workshop
* CLEANUP to delete a workshop
* RESET to reset associated workshop's resource
* PURGE to delete workshop and reset associated workshop's resource

**CREATE subtasks:**

* **6.** It prepares any infrastructure that might be required for the workshop (Virtual Appliance, Virtual Machine, Docker Container, LDAP config, etc.).
* It generates a random Password for the allocated student.
* It deploys the workshop content on the jupyterhub server in the dedicated student home directory (Notebooks files necessary for the workshops).
* **7.** It sends back the confirmation of the deployment of the workshop, along with the student's required details (i.e password), through API Calls to the wod-api-db server.

**8.** The wod-api-db server tables will be updated in the following manner:

* The customers table shows an active status for the participant row. The password field has been updated.
* The workshops table also gets updated. The capacity field decrements the number of available seats.
* The students tables gets updated as well by setting the allocated student to active

**9.** The wod-api-db server sends the second email to each participant providing them with the details to connect to the workshop environment.

## The Run Phase

**10.** From the email, the particpant clicks on the start button. It will open up a browser to the JupyterHub server and directly open the readme first file of the notebook, presenting the workshop's flow.

Participants will go through the different steps and labs of the workshop connecting to the necessary endpoints and leveraging the different kernels available on the JupyterHub server.

Meanwhile, the wod-api-db server will perform regular checks on how much time has passed. Depending on the time allocation (from 2 to 4 hours) associated with the workshop, the wod-api-db server will send a reminder email usually an hour before the end of the time allocated. The time count actually starts when participants hit the register for the workshop button. It is mentioned in the terms and conditions.

Finally, when the time is up, the wod-api-db server sends a new order to the wod-backend to perform either CLEANUP and/or RESET action for the dedicated studentid.

**RESET subtasks:**

* It resets any infrastructure that was required for the workshop (Virtual Appliance, Virtual Machine, Docker Container, LDAP config, etc..).
* It generates a random password for the allocated student.
* It deletes the workshop content on the JupyterHub server in the dedicated student home directory (Notebooks files necessary for the workshop).
* It sends back the confirmation of the CLEANUP or RESET of the workshop along with the student details (i.e password) through API Calls to the wod-api-db server.

The wod-api-db server tables will be updated in the following manner:

* The customers table will show an inactive status for the participant row. The password field has been updated.
* The workshops table gets also updated. The capacity field increment the number of available seats.
* The students tables gets updated as well by setting the allocated student to inactive.
* The wod-api-db server sends the final email to the participant.

## The React and Reward Phase

* A final email thanks the students for their participation. It provides a link to an external survey and encourages the participants to share their achievement [badge](ADMIN-GUIDE.md#Badges) on social media like linkedin or twitter.

  Et voila!

# Deploying the infrastructure

The overall infrastructure can run on physical servers or Virtual Machines. We usually designate one server for the wod-frontend, one for the wod-api-db and a third (or more) server for the wod-backend.

![WoD Infrastucture](img/wod-infra-open-source.png "WoD Infrastucture")

The project is split into multiple repositories. The project admins will need to decide whether they are willing to develop and propose public-only content to the participants (in which case, the standard environment is sufficient) or add any proprietary and private content (in which case a bit more work is required).

We will start with the simpliest scenario: A public-only approach. Then we will dive into the specificities related to the private approach, which will require the reading of the public-only approach anyway.

## Public-only Deployment: No private wod-backend nor private workshops

**Important Note:**  *This part is compulsory for any type of deployment, public only or public+private.*

### wod-backend server preparation

The installation process is handled by a dedicated repo : [wod-install](https://github.com/Workshops-on-Demand/wod-install). This repo needs to be cloned on every single machine  constituting the WoD architecture. Before cloning the `wod-install` repository, you will need to prepare the server that will host the wod-backend features. When ready, you will proceed with the cloning and then the installation process.

#### Prerequesites

In order to setup the wod-backend server, you will need:

* A fresh minimal OS install on physical/virtualized server running Ubuntu 24.04 leveraging any deployment mechanism of your choice.(e.g. through [iLO](https://en.wikipedia.org/wiki/HPE_Integrated_Lights-Out), automatic installation with [preseed](https://wiki.debian.org/DebianInstaller/Preseed), [virt-lightning](https://github.com/virt-lightning/virt-lightning), etc...).
* A Linux account with sudo priviledges on your Linux distro. Name it `install`

**Note:** *In order to support 100 concurrent users, you need:*

* 2 cpus or more machine
* 128 GB of RAM
* 500 GB of storage

We are currently using an either an ProLiant DL360 Gen10 server on our different production sites or QEMU/VMWare VMs on test sites.

When done with OS installation and preparation

* From the wod-backend server (aka JupyterHub server), as the `install` user, you will need to clone the `wod-install` repo first.

```shellsession
git clone https://github.com/Workshops-on-Demand/wod-install.git
cd wod-install/
```

* Examine default installation parameters and adapt when necessary accordingly. Files are self-documented.

Look at the following file `ansible/group_vars/all.yml`

```shellsession
cat ansible/group_vars/all.yml
---
# We create fixed user accounts to provide an isolated execution environment to run the jupyter notebooks
# They are called studentXXX where XXX is comprised between WODUSERMIN and WODUSERMAX defined below poentially with the addition of an offset (UIDBASE) for their uid/gid
# Their home directory is located under /student and is thus named /student/studentXXX
# Corresponding JupyterHub accounts are also created
#
# WODUSERMIN indicates the starting ID of the Linux and Jupyter user account range
#
WODUSERMIN: 1
#
# WODUSERMAX indicates the ending ID of the Linux and Jupyter user account range
#
WODUSERMAX: 100
#
# Branding management - Use if you want to customize Logo and Notebooks branding
#
BRANDING: "WoD Developer"
BRANDINGWOD: "WoD Developer"
BRANDINGLOGO: "![WoDlogo](img/logo.png)"
BRANDINGLOGOURL: "![WoDlogo](img/logo.png)"
BRANDINGURL: "https://wod.io"
BRANDINGSLACK: ""
BRANDINGX: ""
#
# Survey management - Use if you want to ask for feedbacks on your Workshops - Look at existing conclusion notebooks
SURVEYURL: TBD
SURVEYCHALURL: TBD
#
# You may want to use these variables if you have an OPNSense server as a security FW and allowing http comm internally
#
#OPNSENSEKEY:
#OPNSENSESEC:
#OPNSENSEIP:
#OPNSENSEPORT:
```
See the example below for a wod-backend server.

### wod-backend server installation

The installation is based on a common install script [install.sh](https://github.com/Workshops-on-Demand/wod-install/blob/main/install/install.sh) that allows the deployment of the different servers comprising the solution. The script is located under the `wod-install/install/` directory.

It can be called as follows:


`install.sh [-h][-t type][-i ip][-g groupname][-b backend[:beport:[beproto]][-n number][-j backendext[:beportext[:beprotoext]]][-f frontend[:feport[:feproto]]][-w frontendext[:feportext[:feprotoext]]][-a api-db[:apidbport[:apidbproto]]][-e api-dbext[:apidbportext[:apidbprotoext]]][-u user][-p postport][-k][-c][-s sender]`

As you can see on the command line, the -t parameter will define whether you install a wod-backend, a wod-api-db, or a wod-frontend server. Having this information, the script will clone the relevant repositoriesy for the installation. If `t=backend`, then the `wod-backend` repository is cloned as part of the installation process, and the relevant installation scripts are called. Same goes for the wod-api-db and the wod-frontend servers.

`-h` provides the help

```shellsession
install@wod-backend2-u24:~/wod-install/install$ sudo ./install.sh -h
install.sh called with -h
install.sh [-h][-t type][-i ip][-g groupname][-b backend[:beport:[beproto]][-n number][-j backendext[:beportext[:beprotoext]]][-f frontend[:feport[:feproto]]][-w frontendext[:feportext[:feprotoext]]][-a api-db[:apidbport[:apidbproto]]][-e api-dbext[:apidbportext[:apidbprotoext]]][-u user][-p postport][-k][-c][-s sender]

where:
-a api-db    is the FQDN of the REST API/DB server
             potentially with a port (default 8021)
             potentially with a proto (default http)
             example: api.internal.example.org
             if empty using the name of the frontend

-b backend   is the FQDN of the backend JupyterHub server,
             potentially with a port (default 8000).
             potentially with a proto (default http)
             if empty uses the local name for the backend
             If you use multiple backend systems corresponding to
             multiple locations, use option -n to give the backend
             number currently being installed, starting at 1.

             When installing the api-db server you have to specify one
             or multiple backend servers, using their FQDN separated
             with ',' using the same order as given with the -n option
             during backend installation.

-e api-dbext is the FQDN of the REST API server accessible externally
             potentially with a port (default 8021)
             potentially with a proto (default http)
             example: api.external.example.org
             if empty using the name of the api-db
             useful when the name given with -a doesn't resolve from
             the client browser

-f frontend  is the FQDN of the frontend Web server
             potentially with a port (default 8000).
             potentially with a proto (default http)
             example: fe.external.example.org
             if empty using the name of the backend

-g groupname is the ansible group_vars name to be used
             example: production, staging, test, ...
             if empty using 'production'

-i ip        IP address of the backend server being used
             if empty, try to be autodetected from FQDN
             of the backend server
             Used in particular when the IP can't be guessed (Vagrant)
             or when you want to mask the external IP returned
             by an internal one for /etc/hosts creation

-j backext   is the FQDN of the backend JupyterHub server accessible externally
             potentially with a port (default 8000).
             potentially with a proto (default http)
             example: jupyterhub.external.example.org
             if empty using the name of the backend
             useful when the name given with -b doesn't resolve from
             the client browser

-k           if used, force the re-creation of ssh keys for
             the previously created admin user
             if not used keep the existing keys in place if any
             (backed up and restored)
             if the name of the admin user is changed, new keys
             systematically re-created

-c           if used, force insecured curl communications
             this is particularly useful for self-signed certificate
             on https services
             if not used keep curl verification, preventing self-signed
             certificates to work

-n           if used, this indicates the number of the backend
             currently installed
             used for the backend installation only, when multiple
             backend systems will be used in the configuration
             example (single backend server install on port 9999):
              -b be.int.example.org:9999
             example (first of the 2 backends installed):
              -b be1.int.example.org:8888 -n 1
             example 'second of the 2 backends installed):
              -b be2.int.example.org:8888 -n 2
             example (install of the corresponding api-db server):
              -b be.int.example.org:8888,be2.int.example.org:8888

-p postport  is the port on which the postfix service is listening
             on the backend server
             example: -p 10030
             if empty using default (10025)

-s sender    is the e-mail address used in the WoD frontend to send
             API procmail mails to the WoD backend
             example: sender@example.org
             if empty using wodadmin@localhost

-t type      is the installation type
             valid values: appliance, backend, frontend or api-db
             if empty using 'backend'

-u user      is the name of the admin user for the WoD project
             example: mywodadmin
             if empty using wodadmin
-w frontext  is the FQDN of the frontend JupyterHub server accessible externally
             potentially with a port (default 8000).
             potentially with a proto (default http)
             example: frontend.external.example.org
             if empty using the name of the frontend
             useful to solve CORS errors when external and internal names
             are different


Full installation example of a stack with:
- 2 backend servers be1 and be2 using port 8010
- 1 api-db server apidb on port 10000 using https
- 1 frontend server front on port 8000
- all declared on the .local network
- internal postfix server running on port 9000
- e-mail sender being wodmailer@local
- ansible groupname being test
- management user being wodmgr

On the be1 machine:
  ./install.sh -a apidb.local:10000:https -f front.local:8000 \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -b be1.local:8010 -n 1 -t backend
On the be2 machine:
  ./install.sh -a apidb.local:10000:https -f front.local:8000 \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -b be2.local:8010 -n 2 -t backend
On the apidb machine:
  ./install.sh -a apidb.local:10000:https -f front.local:8000 \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -b be1.local:8010,be2.local:8010 -t api-db
On the frontend machine:
  ./install.sh -a apidb.local:10000:https -f front.local:8000 \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -t frontend
```

`install.sh` performs the following tasks:

* Calls the `install-system-<< distribution name >>.sh` script which:
  * Updates the Linux distribution package repository.
  * Installs the minimal set of required packages (`ansible, git, jq, openssh server, npm`).
* Creates an admin user as defined upper (default is `wodadmin`) with sudo rights.
* Calls the `install-system-<< type >>.sh` script if it exists (type being one of backend, frontend or api-db.
* Calls the `install-system-common.sh` script that performs the following tasks:
  * Cleanup of previously installed repo (beware if you made local modifications !).
  * Clone required Github repos (leveraging install.repo file) : public Backend and public Private repos.
  * Create ssh keys for `wodadmin` user.
* Calls the `install_system.sh` script that performs the following tasks:
  * Install the necessary stack based on selected type.
  * Create a `wod.sh` script and wod.yml YAML file in the `/etc` directory containing common variables to be used by all other scripts/playbooks.
  * Source the `/etc/wod.sh` file created.
  * Creates the Ansible inventory files.
  * Creates an Ansible YAML file named after the group name under the private ansible directory in the `generated` sub-directory (allowing private modifications without impacting the upstream repository content).
  * Install required Ansible galaxies (`community.general` and `posix`).
  * Call private scipts during the seyup if they exists (seen later in this guide).
  * Cleanup of previously installed jupyterhub (beware if you made local modifications !).
  * Creates an shell script file named wod-private.sh under the private ansible directory (allowing private modifications without impacting the upstream repository content) containing the credentials for the API and DB access.
  * Calls the Ansible playbook `install_<< type >>.yml` which will perform the software installation and configuration of the WoD `<< type >>` server.
  * Calls the Ansible playbook `check_<< type >>.yml` which will perform the compliance checks (also run every day on the server).

At the end of the installation process:

* you will have a JupyterHub server running on port 8000
* You will get a new `wodadmin` user (Default admin)
* You will get a set of 100 students (Default value)

All playbooks are self-documented. Please check for details.

**Note:** *A wod-install.log is available under the home folder of the install user under the `.wod-install` directory. It contains the installation log along with a another file containg the wodadmin credentials.*

We leave it to you to handle the necessary port redirection and SSL certificates management when needed. In our case, we went for a simple yet efficient solution based on an OPNSense Firewall along with a HAProxy setup to manage ports'redirection, HTTP to HTTPS Redirection, SSL Certificates. The wod-backend also includes a Fail2ban service for login security management.

At this point, you should be able to access your JupyterHub environment with a few pre-installed set of kernels like `bash, python, ansible, ssh, PowerShell`.

You can then start developing new notebooks for your public based environment. Look below in this guide for explanations.

If you need to develop private content that cannot be shared with the wider Open Source Community because of dedicated IP, the next section in this article will explain how to handle this.

## **How to handle private-content based Workshops-on-Demand**

### *(private wod-backend + private workshops on top of default public wod-backend and notebooks)*

The principle remains similar, with a few differences explained below.

* After cloning the wod-install repository, you will fork the following public private [repo](https://github.com/Workshops-on-Demand/wod-private.git) on Github under your own Github account (we will refer to it as `Account`).
* Next, clone the forked repo.
```shellsession
git clone https://github.com/Workshops-on-Demand/wod-private.git
cd wod-private
git remote add myrepo https://github.com/Account/wod-private.git
```
* Edit the `ansible/group_vars/all.yml` file to customize your setup. You can create a `<groupname>` directory ander the `ansible` directory in which you can create similar files to the one you have in `wod-backend/ansible/groupname` after the previous installation. This variable `<groupname>` defines possible wod-backend server in your environement. You could have multiple groupnames. In our case, we have defined `sandbox`, `test`, `staging` and several `production` subdirectories, all defining a different wod-backend environment. These files will be used to override the default values specified by the public version delivered as part of the default public installation.
* Commit and push changes to your repo.
* Create an `install.priv` file located in the `wod-install/install` directory when using a private repo (consider looking at [install.repo](https://github.com/Workshops-on-Demand/wod-backend/blob/main/install/install.repo) file for a better understanding of the variables).
* Define the WODPRIVREPO and WODPRIVBRANCH variables as follows:
  * `WODPRIVBRANCH="main"`
  * `WODPRIVREPO="git@github.com:Account/Private-Repo.git wod-private"`

#### When using a token

Please refer to the following [url](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to generate a `token` file in the `install` sub-directory of wod-backend:

* Edit the `install.priv` file located in `install` directory of wod-install:
  * Create a line before variable declaration reading ``token=`cat $EXEPATH/token` ``
  * Use the token in the url `WODPRIVREPO="git clone https://user:$token@github.com/Account/wod-private.git wod-private"` to take advantage of it.

You are now ready to perform the installation again to support a private repository.

Please note that this setup phase can be concurrent with the public setup phase. Indeed, the install scripts should detect the presence of the private repository owing to the presence of the install.priv file. It will automatically adjust the different scripts and variables to add the relevant content. It will actually overload some of the variables with private ones.

Also note that if you use a different branch than the standard one (`main` usually), you can use additional variables in the `install.priv` file to overwrite the branch with the name you use instead. After the clone you'll then point to the right branch.

You now have a working wod-backend server in place. Congratulations! Now we have to look at the lifecycle of the wod-backend server. How does a workshop registration work from the wod-backend server side? How do you manage this server on a daily basis? How and when do you need to update it ? This is explained in the next paragraphs.

# Workshops deployment lifecycle

The following picture depicts what happens on the wod-backend server when a participant registers for a workshop. Upon registration the wod-api-db sends instructions to the wod-backend server through a procmail API call so the latter can proceed with the workshop preparation and deployment. Once these tasks are completed, it provides the wod-api-db server with the relevant information

Let's now look in details what is really happening on the wod-backend server's side:

## The procmail API

This is a mail parsing process allowing the wod-backend server to retrieve the relevant information in order to perform appropriate actions. As with any API, it uses verbs to perform actions. In our case, we leverage CREATE, CLEANUP, RESET and PURGE.

You can get more info on [procmail](https://fr.wikipedia.org/wiki/Procmail) usage, on [this page](https://wiki.archlinux.org/title/Procmail).

Take a look at the following template of the `.procmailrc` file that will be expanded at setup time.

```
cat wod-backend/sys/procmailrc.j2
MAILDIR=$HOME/.mail      # You'd better make sure it exists
DEFAULT=$MAILDIR/mbox
LOGFILE=$MAILDIR/from

:0b
#* ^From.*{{ WODSENDER }}.*
# \/ defines what will be matched in $MATCH
* ^Subject: *CREATE \/[1-9]+.*
| {{ WODSCRIPTDIR }}/procmail-action.sh CREATE $MATCH

:0b
#* ^From.*{{ WODSENDER }}.*
# \/ defines what will be matched in $MATCH
* ^Subject: *CLEANUP \/[1-9]+.*
| {{ WODSCRIPTDIR }}/procmail-action.sh CLEANUP $MATCH

:0b
#* ^From.*{{ WODSENDER }}.*
# \/ defines what will be matched in $MATCH
* ^Subject: *RESET \/[1-9]+.*
| {{ WODSCRIPTDIR }}/procmail-action.sh RESET $MATCH

:0b
#* ^From.*{{ WODSENDER }}.*
# \/ defines what will be matched in $MATCH
* ^Subject: *PURGE \/[1-9]+.*
| {{ WODSCRIPTDIR }}/procmail-action.sh PURGE $MATCH
```

The `From:` header is important as `.procmailrc` checks that the sender is the one configured during the installation (`-s` option of the `install.sh` script). Any mail from any other sender but the configured one is not processed.

This API is actually based on a script `procmail-action.sh`, which defines the different operations linked to the verbs passed through the API calls via `.procmailrc`.

## CREATE Verb

Let's start with a CREATE scenario looking at the very first lines of the procmail log file (in `~wodadmin/.mail/from`).
![WoD backend create lifecycle](img/wod-backend-create.png "WoD backend create lifecycle")

```
From xyz@hpe.com<mailto:xyz@hpe.com>  Wed Mar  1 15:10:41 2025
Subject: CREATE 401 825 frederic.passeron@hpe.com
Folder: /home/wodadmin/wod-backend/scripts/procmail-action.sh CREATE       14
```

In the `Subject:` header, look for the API verb CREATE followed by the 3 parameters student id (401), participant id (825) and finally the registered participant email (frederic.passeron@hpe.com). These 3 parameters are passed to the `procmail-action.sh` script after the verb for processing.

In order to work properly, `procmail-action.sh` sources 4 files:

1. `/etc/wod.sh` which sets a large number of variables: This script is generated at install time.
```
cat /etc/wod.sh
# Shell variables for WoD
#
#
# This is the wod.sh script, generated at install
# Please do not modify directly
#
# Name of the admin user
export WODUSER=wodadmin

# Name of the wod machine type (backend, api-db, frontend, appliance)
export WODTYPE=backend

# Name of the api-db server
export WODAPIDBFQDN="wod-apidb1.hpedev-floss-wod.lab"
# Port of the api-db server
export WODAPIDBPORT="8021"
# Name of the external api-db server
export WODAPIDBEXTFQDN="wod-apidb1.hpedev-floss-wod.lab"
# Port of the external api-db server
export WODAPIDBEXTPORT="8021"
# Combined URL for API access
export WODAPIDBURL=http://wod-apidb1.hpedev-floss-wod.lab:8021/api
# Combined URL for external API access
export WODAPIDBEXTURL=http://wod-apidb1.hpedev-floss-wod.lab:8021/api
# Combined URL for API backend access
export WODBEAPIURL=http://wod-backend1.hpedev-floss-wod.lab:8000
#
# Places where repos have been exported for WODUSER
export WODROOTDIR=/home/wodadmin
#
# INSTALL PART
# The install dir has some fixed subdirs  for shared content
# wod-install (WODINSDIR)
#    |---------- ansible (WODINSANSDIR)
#    |---------- scripts (WODINSSCRIPTDIR)
#    |---------- skel
#    |---------- sys (WODINSSYSDIR)
#
export WODINSDIR=/home/wodadmin/wod-install/install
export WODINSANSDIR=/home/wodadmin/wod-install/ansible
export WODINSSYSDIR=/home/wodadmin/wod-install/sys
export WODINSSCRIPTDIR=/home/wodadmin/wod-install/scripts
export WODANSIBLEDIR=/home/wodadmin/wod-backend/ansible
#
export WODGROUP=test
#
# BACKEND PART
# The backend dir has some fixed subdirs
# wod-backend (WODBEDIR)
#    |---------- ansible (WODANSIBLEDIR)
#    |---------- conf
#    |---------- scripts (WODSCRIPTDIR)
#    |---------- skel
#    |---------- sys (WODSYSDIR)
#
# Location of the backend directory
#
export WODBEDIR=$WODROOTDIR/wod-backend

# PRIVATE PART
# These 3 dirs have fixed names by default that you can change in this file
# they are placed as sister dirs wrt WODBEDIR
# This is the predefined structure for a private repo
# wod-private (WODPRIVDIR)
#    |---------- ansible (WODANSIBLEPRIVDIR)
#    |---------- notebooks (WODPRIVNOBO)
#    |---------- scripts (WODSCRIPTPRIVDIR)
#
export WODPRIVDIR=$WODROOTDIR/wod-private
export WODANSIBLEPRIVDIR=$WODPRIVDIR/ansible
export WODSCRIPTPRIVDIR=$WODPRIVDIR/scripts
export WODSYSPRIVDIR=$WODPRIVDIR/sys
export WODPRIVNOBO=$WODPRIVDIR/notebooks
WODPRIVINV=""
# Manages private inventory if any
if [ -f $WODPRIVDIR/ansible/inventory ]; then
    WODPRIVINV="-i $WODPRIVDIR/ansible/inventory"
fi
export WODPRIVINV

# AIP-DB PART
#    |---------- ansible (WODANSIBLEDIR)
#    |---------- conf
#    |---------- scripts (WODSCRIPTDIR)
#    |---------- sys (WODSYSDIR)
#
export WODAPIDBDIR=$WODROOTDIR/wod-api-db

# FRONTEND PART
#    |---------- ansible (WODANSIBLEDIR)
#    |---------- conf
#    |---------- scripts (WODSCRIPTDIR)
#    |---------- sys (WODSYSDIR)
#
export WODFEDIR=$WODROOTDIR/wod-frontend
export WODNOBO=$WODROOTDIR/wod-notebooks
#
# DERIVED DIRS valid on the node being installed
export WODSYSDIR=$WODROOTDIR/wod-$WODTYPE/sys
export WODSCRIPTDIR=$WODROOTDIR/wod-$WODTYPE/scripts

# These dirs are also fixed by default and can be changed as needed
export WODSTUDDIR=/student
#
export WODANSPRIVOPT=" -e @/home/wodadmin/wod-private/ansible/group_vars/all.yml -e @/home/wodadmin/wod-private/ansible/generated/test"
export WODANSPLAYOPT="-e WODGROUP=test -e WODUSER=wodadmin -e WODBEDIR=/home/wodadmin/wod-backend -e WODNOBO=/home/wodadmin/wod-notebooks -e WODPRIVNOBO=/home/wodadmin/wod-private/notebooks -e WODPRIVDIR=/home/wodadmin/wod-private -e WODAPIDBDIR=/home/wodadmin/wod-api-db -e WODFEDIR=/home/wodadmin/wod-frontend -e WODSTUDDIR=/student -e WODANSIBLEDIR=/home/wodadmin/wod-backend/ansible -e WODANSIBLEPRIVDIR=/home/wodadmin/wod-private/ansible -e WODSCRIPTDIR=/home/wodadmin/wod-backend/scripts -e WODSCRIPTPRIVDIR=/home/wodadmin/wod-private/scripts -e WODSYSDIR=/home/wodadmin/wod-backend/sys -e WODSYSPRIVDIR=/home/wodadmin/wod-private/sys -e WODINSDIR=/home/wodadmin/wod-install/install -e WODINSSCRIPTDIR=/home/wodadmin/wod-install/scripts -e WODINSANSDIR=/home/wodadmin/wod-install/ansible -e WODINSSYSDIR=/home/wodadmin/wod-install/sys -e @/home/wodadmin/wod-install/ansible/group_vars/all.yml -e WODLDAPSETUP=0 -e WODAPPMIN=0 -e WODAPPMAX=0"
export WODUSERMAX=100
```

2. `wod-private/scripts/wod-private.sh` which contains credentials to access the API service and the PostgreSQL DB.
```
cat wod-private/scripts/wod-private.sh
export WODAPIDBUSER=""
export WODAPIDBUSERPWD=""
export WODAPIDBADMIN=""
export WODAPIDBADMINPWD=""
export WODPGDB=""
export WODPGUSER=""
export WODPGPASSWD=""
```

3. `wod-backend/scripts/random.sh` which contains the randomly generated password for the user
```
export RANDOMPW=""
```

4. `wod-backend/scripts/functions.sh`which is a library of shell functions used by many scripts, among which can be found procmail-action.sh. Details are shown below in alphabetic order:

- `wod_cleanup_processes_student()` This functions kills all remaining processes from the given student id.Used during CLEANUP phase.
- `wod_create_var_passwd()` This function creates a variable file in which password is stored from the ansible playbook with $randompw
- `wod_erase_student()` This function erases all the content from the allocated student's home directory. You want to make sure that the home directory is not compromised. You want to start clean. Used during CLEANUP phase.
- `wod_generate_randompwd() This function creates a random password for a user. It is used both for local and LDAP users' passwords. If the workshop requires an LDAP authentication (wod_get_ldap_status() functions will return this information) then another function is used to update the LDAP server with the password for the given student (wod_update_ldap_passwd()`)
The generated password will be sent back to the wod-api-db server so that it can then send an email to allow participant to connect to the workshop.
- `wod_get_appliance_baseip()` This function returns the base IP address of the appliance related to the workshop given as parameter
- `wod_get_cleaned_status()` This function returns the status of the cleaned boolean for the user id given as parameter
- `wod_get_compile_status()` This function will check whether the workshop needs some scripts to be compiled. For instance, if you need to authenticate against a private cloud portal and you don't want your participants to see the credentials, make sure to check the relevant box in the workshop table of the database. This compile feature will compile the authentication scripts into an executable that cannot be edited.
- `wod_get_completion_ratio()` This function calculates a completion ratio at the end of a workshop. Used during CLEANUP phase (not CREATE). Allows to retrieve information through a simple computing of the numbers of notebooks cells executed thoughout the different exercices of the workshop a ratio. Gives a percentage on how much of the workshop is actually run. Participants are asked to fill out a form in a conclusion notebook which is present in every student's workshop's folder.
- `wod_get_ldap_status()` Does the workshop require LDAP authentication?
- `wod_get_monoappliance_status()` This function returns the status of the mono-appliance boolean for the workshop id given as parameter (most cases)
- `wod_get_multiappliance_status()` This function returns the status of the multi-appliance boolean for the workshop id given as parameter (only when your workshop needs multiple appliances to work)
- `wod_get_range_max()` This function returns the max of the range of the users booked for the workshop id given as parameter
- `wod_get_range_min()` This function returns the min of the range of the users booked for the workshop id given as parameter
- `wod_get_reset_status()` This function returns the status of the reset boolean for the workshop id given as parameter. A reset should be performed when an appliance requires a reset script to be executed for final cleanup.
- `wod_get_session_token()` This function retrieves the necessary token to make an API call to the wod-api-db server.
- `wod_get_varpass_status()` Does the workshop require the use of the student password as a variable ?
- `wod_get_workshop_id()` From the workshop name, it retrieves the workshop's ID from the wod-api-db server.
- `wod_get_workshop_name()` This function extracts the workshop name from the mail body, for example, WKSHP-API101 (They have to start with the WKSHP- prefix and are uppercase)
- `wod_test_stdid()` This function checks the student ID provided by procmail API is valid: For each workshop, a dedicated student range is allocated. This function exits when the student ID is not in the correct range.
- `wod_update_ldap_passwd()` This function updates the LDAP passwd for the student under management using the student id
- `wod_wait_till_clean()` This function waits in order to avoid concurrency with the CLEANUP phase

`procmail-action.sh` then calls the necessary functions and scripts to perform the CREATE operation (and following verbs CLEANUP, RESET, PURGE).

- If an appliance is needed for the workshop, then the following script is called: `create-<WKSHP>.sh`. This will prepare the appliance (deploying a docker image on it for instance) and setup user env on the appliance accordingly (ssh keys, skeletons)

For instance, [create-WKSHP-ML101.sh](https://github.com/Workshops-on-Demand/wod-backend/blob/main/scripts/create-WKSHP-ML101.sh) will perform the following tasks in order to prepare the appliance for the workshop: It will start by reseting the appliance with the `reset-<WKSHP>.sh` script. Then, it calls a second script aiming at preparing a generic appliance `create-appliance.sh`. Once done with these two, it moves on with the proper customization of the appliance for the given student.

```
cat create-WKSHP-ML101.sh.j2
#!/bin/bash

set -x

source {{ WODSCRIPTDIR }}/functions.sh

# Start by cleaning up stuff - do it early as after we setup .ssh content
{{ WODSCRIPTDIR }}/reset-$ws.sh
{{ WODSCRIPTDIR }}/create-appliance.sh

NAME=mllab
TMPDIR=/tmp/$NAME.$stdid


mkdir -p $TMPDIR

# Define local variables
echo wid=$wid
WODAPPMIN=`wod_get_range_min $wid`
echo stdid=$stdid
echo WODAPPMIN=$WODAPPMIN
mlport=$(($stdid-$WODAPPMIN+{{ hostvars[inventory_hostname]['MLPORT-WKSHP-ML101'] }}))
mlport2=$(($stdid-$WODAPPMIN+{{ hostvars[inventory_hostname]['MLPORT2-WKSHP-ML101'] }}))
httpport=$(($stdid-$WODAPPMIN+{{ hostvars[inventory_hostname]['HTTPPORT-WKSHP-ML101'] }}))

cat > $TMPDIR/dockerd-entrypoint.sh << EOF
export HTTPPORT
tini -g -- start-notebook.sh &
sleep 3
jupyter lab list | tail -1 | cut -d'=' -f2 | cut -d' ' -f1 > {{ WODSTUDDIR }}/student$stdid/mltoken
sleep infinity
EOF

cat > $TMPDIR/Dockerfile << EOF
FROM ${NAME}:latest
USER root
COPY dockerd-entrypoint.sh /usr/local/bin/
ENTRYPOINT /usr/local/bin/dockerd-entrypoint.sh
RUN mkdir -p {{ WODSTUDDIR }}/student$stdid
RUN useradd student$stdid -u $stdid -g 100 -d {{ WODSTUDDIR }}/student$stdid
RUN chown student$stdid:users {{ WODSTUDDIR }}/student$stdid
# Unlock the account
RUN perl -pi -e "s|^student$stdid:!:|student$stdid:\$6\$rl1WNGdr\$qHyKDW/prwoj5qQckWh13UH3uE9Sp7w43jPzUI9mEV6Y1gZ3MbDDMUX/1sP7ZRnItnGgBEklmsD8vAKgMszkY.:|" /etc/shadow
# In case we need sudo
#RUN echo "student$stdid   ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers
WORKDIR {{ WODSTUDDIR }}/student$stdid
USER student$stdid
ENV NB_USER student$stdid
ENV NB_UID $stdid
ENV HTTPPORT $httpport
RUN git clone https://github.com/snowch/ml-101 {{ WODSTUDDIR }}/student$stdid/
RUN /opt/conda/bin/jupyter-nbconvert --clear-output --inplace {{ WODSTUDDIR }}/student$stdid/*.ipynb
EOF


# Look at https://stackoverflow.com/questions/34264348/docker-inside-docker-container
# and http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/
# For security consider using https://github.com/nestybox/sysbox
cat > $TMPDIR/docker-compose.yml << EOF
version: '3.5'
services:
  $NAME$stdid:
    image: $NAME$stdid
    build: .
    #privileged: true
    ports:
      - "$httpport:8888"
      - "$mlport:4040"
      - "$mlport2:4041"
#    volumes:
#      - /var/run/docker.sock:/var/run/docker.sock
EOF
cat > $TMPDIR/launch-$NAME << EOF
#!/bin/bash
cd $TMPDIR
docker compose up --build -d
EOF

# Copy content and start container
ssh -q {{ NOCHECKSSH }} $WODRTARGET mkdir -p $TMPDIR
scp -q {{ NOCHECKSSH }} $TMPDIR/Dockerfile $TMPDIR/docker-compose.yml $TMPDIR/launch-$NAME $TMPDIR/dockerd-entrypoint.sh $WODRTARGET:$TMPDIR
ssh -q {{ NOCHECKSSH }} $WODRTARGET chmod 755 $TMPDIR $TMPDIR/dockerd-entrypoint.sh
ssh -q {{ NOCHECKSSH }} $WODRTARGET chmod 644 $TMPDIR/docker-compose.yml $TMPDIR/launch-$NAME $TMPDIR/Dockerfile
ssh -q {{ NOCHECKSSH }} $WODRTARGET sudo su - student$stdid -c \'bash $TMPDIR/launch-$NAME\'

# ssh to get token from running container
container_id=$(ssh -q {{ NOCHECKSSH }} $WODRTARGET docker ps -aqf "name=$NAME$stdid")
# We need time to let jupyterhub start before getting token
sleep 5
ssh -q {{ NOCHECKSSH }} $WODRTARGET "docker cp $container_id:{{ WODSTUDDIR }}/student$stdid/mltoken $TMPDIR"
scp -q {{ NOCHECKSSH }} $WODRTARGET:$TMPDIR/mltoken $TMPDIR

# ansible-vault to store it in the pass file creating a MLTOK ansible variable
if [ ! -d "{{ WODANSIBLEPRIVDIR }}/{{ WODGROUP }}/pass" ]; then
        mkdir -p "{{ WODANSIBLEPRIVDIR }}/{{ WODGROUP }}/pass"
fi
# We can overwrite the existing variables file as we don't need PASSSTU
ansible-vault encrypt_string --vault-password-file "{{ WODANSIBLEPRIVDIR }}/vault_secret" `cat $TMPDIR/mltoken` --name "'MLTOK'" > "{{ WODANSIBLEPRIVDIR }}/{{ WODGROUP }}/pass/variables_${w}_${stdid}.yml"
ansible-vault encrypt_string --vault-password-file "{{ WODANSIBLEPRIVDIR }}/vault_secret" $httpport --name "'HTTPPORT'" >> "{{ WODANSIBLEPRIVDIR }}/{{ WODGROUP }}/pass/variables_${w}_${stdid}.yml"

# Cleanup
# ssh -q {{ NOCHECKSSH }} $WODRTARGET sudo rm -rf $TMPDIR
rm -rf $TMPDIR
```

- The `ansible/copy_folder.yml` playbook is now executed to deploy the notebooks and scripts necessary for the participant to run the workshop. Remember that the participant got a student (with a dedicated student id. For instance: student41) allocated to him/her at the time of the registration. This student id is picked from a range that is allocated for the workshop. The admin decides on the maximum capacity it allocates to a given workshop. This playbook is historically one of very first playbooks we used and therefore a very important one. It performs the necessary actions to deploy and personnalize (by substituting Ansible variables) the selected notebook to the appropriate student home folder.

- In the certain cases, some post deployment actions are needed. For instance, you may want to `git clone` some repository to leverage some data stored there. This can only occur when done with the deployment. Therefore, a `post-copy-<WKSHP>.sh` is called.

- Finally, the workshop is now ready to be used by the participant. The wod-backend server therefore needs to inform the wod-api-db server of this. To do so, it will perform two API calls:
  *   The first API call will update the password data for the participant's allocated student.
  *   The second API call will update the participant's allocated student's status to active.

These changes will trigger the wod-api-db server to send a second email to the participant. This email will contain the necessary information for the participant to connect to its notebooks environment. The participant will then run the workshop. For each workshop, a dedicated time window is allocated. Some workshops will take longer to be run than others. The time windows varies from 2 to 4 hours maximum.The system knows how to set it up so that it will time out. This means that once the participant hits the register button on the wod-frontend web portal, the clock starts ticking.

Some background checks take place on the web portal to verify time spent since the registration to a given workshop. As a consequence, a reminder email is sent an hour before the workshop times out. When the bell rings at the end of the class, a new procmail API call is made to the wod-backend server ordering a CLEANUP action. The participant can also trigger this action by registering to a new workshop before the end of the current one. He/She will have to provide the necessary information to the wod-frontend web portal in order to end the current workshop.

## CLEANUP Verb
Let's see what is happening on the wod-backend server to perform this CLEANUP verb.
![WoD backend cleanup lifecycle](img/wod-backend-cleanup.png "WoD backend cleanup lifecycle")

As you can see, it does not differ much from the CREATE. We still need to gather data to interact with the proper workshop from the right student. The `.procmailrc` is providing us with this information. Then, the automation kicks in through the `procmail-action-sh` script.

What differs is linked to specific end of job actions that need to be performed. So CLEANUP does:
- The computation of the completion ratio.
- An API call to send the completion ratio figure to the wod-api-db server. This can be later queried to build up reporting dashboard.
- Now that we have extracted the necessary data from the student's notebooks, we can perform a cleanup of the student folder. (see `wod_erase_student()` upper).
- On top of cleaning up the student folder, we also kill all the allocated student's processes. (see `wod_cleanup_processes_student()` upper).
- If any appliance is involved, the script calls the related `cleanup-<WKSHP>.sh` that will perform the necessary cleanup processes on the appliance.
- Finally, just like for the creation of a workshop process, we need to tell the wod-api-db server that the cleanup is now done. Therefore, several API calls are made to update tables in the database. We also generate a new password for the student at the cleanup phase to prevent unregistered logins and record it. The student status is set to inactive. The capacity figure is incremented by one to make the seat available again.

As for the CREATE phase, the regular checks occuring on the wod-api-db server will get these data and trigger the final email sent to the participant, thanking him for his particpation.

## RESET Verb
Now let's look at the RESET verb.
![WoD backend reset lifecycle](img/wod-backend-reset.png "WoD backend reset lifecycle")

You may wonder about the differences between the CLEANUP and RESET verbs. CLEANUP only takes care of students whereas RESET takes care of a larger scope.

When a CLEANUP occurs, it deals with the participant's student workshop and home directory (the workshop directory belonging to the home directory). It cleans up workshop content, ssh keys, skeletons. The RESET will delete leftovers from the workshop's exercices. For instance, when one runs the [Docker 101](https://github.com/Workshops-on-Demand/wod-notebooks/tree/main/WKSHP-Docker101) workshop, Docker images are created, as well as containers on a dedictaed appliance, and they should be removed at the end of the workshop, with some `docker rm/rmi` commands to clean up everything. However, some participants may forget to run these clean up steps themselves as part of the workshop. And the admin needs to make sure that the next participant who will be assigned the same student environment gets a fresh one. Therefore, some measures have to be taken. These measures take place when a reset flag is associated to the workshop in the database. This is typically mandatory when dealing with appliances.

During the CLEANUP phase, a check is actually performed to test the presence of this flag through a simple API call on the wod-api-db server. If the workshop has a reset flag then a dedicated `reset-<WKSHP>.sh` script is called and performs the necessary tasks. In the case of [Docker 101](https://github.com/Workshops-on-Demand/wod-backend/blob/main/scripts/reset-WKSHP-Docker101.sh.j2), it will wipe out any docker images or container or network left over by the student. In some other cases, it will launch a revert to snapshot script on a virtual machine, or anything that makes the context idempotent.

## PURGE Verb
Finally, let's consider the PURGE verb.
![WoD backend purge lifecycle](img/wod-backend-purge.png "WoD backend purge lifecycle")

In a perfect world, we would have covered here what one would somehow expect from any kind of API (GET, PUT, DELETE = CREATE, CLEANUP and RESET). But, this is unfortunately not the case. Even though we did our best to harden the deployment automation, failures might occur. Issues could occur at many different levels. From a backhoe loader cutting an internet line on the very morning of a starting event (preventing you from accessing your remote labs) to unplanned power cuts, or misconfigured power redundancy in PDUs assignment, there are many examples possible of human factor related issues. As a result, the Jupyterhub server or an appliance might become unreachable, and the automation of the workshop's deployment might fail.

In these very cases, you need to be able to cleanup the mess quickly.
![WoD backend failure lifecycle](img/wod-backend-failure.png "WoD backend failure lifecycle")

  *   wod-frontend <-> wod-backend communication issues
  *   JupyterHub server failure
  *   JupyterHub server <-> appliance server communication issues

The PURGE scenario is therefore triggered on Workshops-on-Demand' deployment failures.

At the registration time, when the participant hits the register button on the wod-frontend web portal, an entry is automatically created in the database for him. It associates the participant to a student and a workshop. It also registers the date and start time of the workshop, sets the participant status to 'welcome' in the database and a first email is sent to the participant from the wod-frontend web portal welcoming him to the Workshop-on-Demand and stating to him that within a few minutes a second email will be sent along with the necessary information (credentials and url) to connect to the workshop's environment.

If for any reason, the deployment of the workshop fails and as a consequence, no API call is made back to the wod-api-db from the wod-backend, the wod-api-db could remain stuck forever and so would the participant. To overcome this, we implemented a check on the wod-api-db server to test this welcome status. In a normal scenario, this welcome status gets updated within less than 3 minutes. So if the status is not updated within 10 minutes, we consider that something went wrong during the deployment and as a result, a PURGE scenario is initiated to clean up both the wod-backend and the wod-api-db sides of the related registration. Of course, depending of the wod-backend sanity, some actions could also fail. But from our experience, both wod-api-db and wod-backend are really reliable.

Considering now the most common case: the wod-backend and wod-api-db servers can communicate but the JupyterHub server has issues to communicate with appliances. In terms of tasks associated to the PURGE scenario, you can see that we kept the minimal as there should not be much to clean up on the wod-backend server. Simply consider that it is a CLEANUP scenario without any workshop deployment.

We call the same tasks to begin with as we still need student ID and workshop ID. We then initiate :
- An update of the student's password for security reasons. (See `wod_generate_randompwd()`).
- A cleanup of the student folder. (See `erase-student()`).
- API calls to update tables in the database. The new student password is recorded. The student status is set to inactive. The capacity figure is incremented by one to make the seat available again.

An email is then sent to the participant explaining to him that we encountered an issue with the deployment and that we apologize for this.

Now, you should have a clearer view of what is really happening in the background when one registers for a workshop. Let's cover the day to day operations of the wod-backend.

# Workshops maintenance

After describing the installation, automation and usage of the wod-backend server managing the Jupyter Notebooks, let's describe the management of this wod-backend server, often referred to as Day2 operations.

Once up and running, the main purpose of the wod-backend server is to deliver Workshops-on-Demand. But to do so, it may require updates, upgrades, and/or new kernels for the JupyterHub server. If new workshops are created, this means you'll need new [Jinja templates](https://jinja.palletsprojects.com/en/stable/) for related workshops' scripts (i.e `create-<WKSHP>.sh`, `cleanup-<WKSHP>.sh`, `reset-<WKSHP>.sh`, among others). This also means new variable files. And obviously, these templates and variables will need to be taken into account by scripts and notebooks. Some tasks handle all of this.

## wod-backend server management

If you take a look at the file structure of the `wod-backend` directory, you will discover that the team did its best to sort things properly depending on their relationship to system or workshops.

### Content of the wod-backend server

Simple tree view of the wod-backend directory:

![Tree view of wod-backend directory](img/wod-backend-tree-dot.png "Tree view of wod-backend directory")

The `ansible` folder contains all the necessary playbooks and variables files to support the main functions of the wod-backend server. It provides playbooks for an installation of the wod-backend server itself or for appliances (virtual machines or containers) managed by it, or workshops as well as maintenance tasks.

The `conf` folder hosts configuration files in a Jinja format. Once expanded, the resulting files will be used by relevant workshops.

The `scripts` folder hosts script files used to manage the workshops and appliances lifecycle, as well as the major scripts seen in the deployment of workshops chapter upper.

The `sys` folder hosts script files used during the install of the wod-backend itself.

At the root of the `ansible` directory can be found:

![Tree view of wod-backend/ansible directory](img/wod-backend-tree-ansible.png "Tree view of wod-backend/ansible directory")

`check_backend.yml playbook`: This playbook is used to perform daily compliance checks of the system. These checks ensure that this is a compliant WoD system by checking security (firewall rules, users setup, ...) and services, among other things.

`copy_folder.yml`: Historically, this is one of the very first playbooks we used. It performs the necessary actions to deploy and personnalize (by substituting Ansible variables) the selected notebook to the appropriate student home folder.

`compile_scripts.yml`: Should you need to hide from the student a simple api call that is made on some private endpoint with non-shareable data (credentials for instance), this playbook will make sure to compile it and create an executable file allowing it to happen.

`distrib.yml`: This playbook retrieves the distribution name and version from the machine it is run on.

`install_backend.yml`: This playbook takes care of installing the required packages needed by this server.

`setup_*.yml`: There are several types of setup playbooks in this directory.

* `setup_<WKSHP>_appliance.yml`: These playbooks are responsible for preparing a base appliance for a given workshop by adding and configuring the necessary packages or services related to the workshop.
* `setup_appliance.yml`: This playbook is used to perform the base setup for a JupyterHub environment server or appliance. It includes `setup_base_appliance.yml` playbook.
* `setup_base_appliance`: This takes care of setting the minimal requirements for an appliance. It includes `install_base_system.yml` playbook. On top of it, it creates and configures the necessary users.
* `setup_[docker|podman]_based_appliance.yml`: It performs setup tasks to enable Docker|Podman on a given appliance.

It also hosts the `inventory` file describing the role of JupyterHub servers. Place your server (FQDN) in a group used as PBKDIR namerole (given at install time to differentiate potential multiple installations).

At the root of the `scripts` directory can be found:

![Tree view of wod-backend/scripts directory](img/wod-backend-tree-scripts.png "Tree view of wod-backend/scripts directory")

When one creates a workshop, one needs to provide a series of notebooks and in some cases some scripts to manage the creation and setup of a related appliance along with additional scripts to manage its lifecycle in the overall Workshops-on-Demand architecture (CREATE, CLEANUP, RESET verbs documented earlier). These scripts need to be located in the `scripts` folder. On the other hand, the system scripts are located in the `sys` folder:

![Tree view of wod-backend/sys directory](img/wod-backend-tree-sys.png "Tree view of wod-backend/sys directory")

This directory hosts important scripts and data files for both the system and JupyterHub. You can see for instance `fail2ban` configuration files. Some Jinja templates are present here, too. These templates will be expanded through the `wod-deliver` mechanism allowing the creation of files customized with Ansible variables. All the WoD related tasks are prefixed with `wod` for better understanding and ease of use.

These Jinja templates can refer to some JupyterHub kernel needs like `wod-build-evcxr.sh.j2` that aims at creating a script allowing the rust kernel installation. Some other templates are related to the system and JupyterHub. `wod-kill-processes.pl.j2` has been created after discovering the harsh reality of online mining. In a ideal world, we would not have to explain further as the script would not be needed. Unfortunately, this is not the case. When one offers access to some hardware freely online, sooner or later, he can expect to see his original idea to be hijacked.

`wod-test-action.sh.j2` is another interesting template that will create a script that can be used for testing workshops. This script mimics the procmail API and actually enables you to test the complete lifecycle of a workshop from deployment to cleanup or reset. It is expanded and delivered during every run of the `check_backend.yml` playbook (at install and daily by default).

```
wod-test-action.sh
Syntax: wod-test-action.sh <CREATE|CLEANUP|RESET|PURGE|PDF|WORD> WKSHOP [MIN[,MAX]]
ACTION is mandatory
```

It requires the verb, the workshop's name and the student id. Using the script, one does not need to provide participant id.  The script is run locally on the JupyterHub server as `wodadmin` user.

```shellsession
wod-test-action.sh CREATE WKSHP-API101 121
Action: CREATE
We are working on WKSHP-API101
Student range: 121
Sending a mail to CREATE student 121 for workshop WKSHP-API101
220 server.xyz.com ESMTP Postfix (Ubuntu)
250 2.1.0 Ok
250 2.1.5 Ok
354 End data with <CR><LF>.<CR><LF>
250 2.0.0 Ok: queued as 9749E15403AB
221 2.0.0 Bye
```

In order to retrieve the result of the script, you simply need to run a `tail` command on the log file at `.mail/from`.

```shellsession
tail -1000 .mail/from
....
>From xyz@hpe.com  Fri Mar  3 09:08:35 2025
 Subject: CREATE 121 0
  Folder: /home/wodadmin/wod-backend/scripts/procmail-action.sh CREATE       11
+ source /home/wodadmin/wod-backend/scripts/wod.sh
....
+ echo 'end of procmail-action for student 121 (passwd werty123) with workshop WKSHP-API101 with action CREATE at Fri Mar  3 09:11:39 UTC 2025'
```

The very last line of the trace will provide you with the credentials necessary to test your workshop.

There are two types of activities that can occur on the wod-backend server: punctual or regular. The punctual activity is one that is performed once every now and then. The regular one is usually set up on the wod-backend server as a cron job. Sometimes however, one of these cron tasks can be forced manually if necessary. One of the most important scheduled task is the `wod-deliver` task explained below. Among the punctual tasks is the update of the wod-backend server.

### Update of the wod-backend server

The wod-backend server hosts all the necessary content for delivering workshops: it supplies notebooks, scripts and playbooks to deploy and personalize them. It also hosts some services that are needed by the overall architecture solution (JupyterHub, Procmail, Fail2ban among others).

Services are installed once and for all at the installation time. These services may evolve over time. One may need to update the JupyterHub application to fix a bug or get new features. If you are willing to update these services or add new ones, you will need to update the relevant installation playbooks in the `wod-backend/ansible` directory.

Here is a small extract of the `install_backend.yml` playbook: Full version [here](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/install_backend.yml)

```shellsession
cat ansible/install_backend.yml
- hosts: all
  gather_facts: true
  vars:
    IJAVAVER: "1.3.0"
    KUBECTLVER: "1.21.6"

  tasks:
    - name: Include variables for the underlying distribution
      include_vars: "{{ ANSIBLEDIR }}/group_vars/{{ ansible_distribution }}-{{ ansible_distribution_major_version }}.yml"

    - name: Base setup for a JupyterHub environment server or appliance
      include_tasks: "{{ ANSIBLEDIR }}/setup_base_appliance.yml"

    - name: Add CentOS SC repository into repo list
      become: yes
      become_user: root
      yum:
        name: centos-release-scl-rh
        state: present
      when:
        - ansible_distribution == "CentOS"
        - ansible_distribution_major_version >= "7"

    - name: Add conda GPG Key to APT
      become: yes
      become_user: root
      apt_key:
        url: https://repo.anaconda.com/pkgs/misc/gpgkeys/anaconda.asc
        state: present
      when:
       - ansible_distribution == "Ubuntu"
       - ansible_distribution_major_version >= "20"

      # TODO: Do it for EPEL if really needed
    - name: Add conda APT repository
      become: yes
      become_user: root
      apt_repository:
        repo: deb [arch=amd64] https://repo.anaconda.com/pkgs/misc/debrepo/conda stable main
        state: present
      when:
       - ansible_distribution == "Ubuntu"
```

Possible Use Cases:

* Upgrade to a newer version of JupyterHub
* Add a new kernel to JupyterHub
* Add a new PowerShell library
* Add a new package needed by a workshop.

For e.g:

* Kubectl client
* Terraform client
* PowerShell module
* Python Library

You'll work as the `wodadmin` user in its home directory, under the `wod-backend` subdirectory. As this environment is a clone of the project's repository, the best practice is to fork it (typically on github directly) and commit your changes to it in order to avoid loosing them when the upstream projects makes modifications. We would welcome changes that are beneficial for the overall project of course, feel free to submit pull requests.

In order for your modifications to be taken in account, you'll need to run the `wod-deliver` script manually.

### Regular maintenance of the wod-backend server

On a daily basis, some tasks are launched through this same `wod-deliver` script in order to check the integrity of the wod-backend server. Some tasks are related to the security and integrity of the system. The following playbook is at the heart of this verification: `wod-backend/ansible/check_backend.yml`. Full version of the file is available [here](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/check_backend.yml) for review.

It checks a quite long list of items like:
* `nproc` hard and soft limits
* `nofile` hard and soft limits
* Some `sysctl` params
  * `net.ipv4.tcp_keepalive_time`, value: "1800"
  * `kernel.threads-max`, value: "4096000"
  * `kernel.pid_max`, value: "200000"
  * `vm.max_map_count`, value: "600000"
* Setup UDP and TCP firewall rules
* Enable services:
  * `firewalld` on Rocky based systems
  * `ntp`
* Students (Linux users) Management:
  * Ensure limits are correct for students accounts
  * Copy the skeleton files under `/etc/skel`
  * Test `.profile` file
  * Ensure `vim` is the default EDITOR (of course :-)
  * Setup `logind.conf`
  * Manage `/etc/hosts` file
  * Install the packages update script
  * Setup `cron` for daily package update
  * Deliver create/reset/setup scripts as ansible template for variable expansion
  * Install utility scripts
  * Deliver the system scripts (such as `cleanup-processes.sh.j2`)
  * Installation of the cleanup-processes service
  * Setup weekly cleanup processes task
  * Enable the WoD service
  * Ensure dedicated students groups exists
  * Ensure students (Linux users) exist with their home directory
  * Setup ACL for students wrt JupyterHub configuration
  * Setup default ACL for students wrt JupyterHub configuration
  * Test private tasks YAML file
  * Call private tasks if available. It performs the private part before users management to allow interruption of the deliver script during normal operations - waiting till end of users management can take hours for 2000 users. Potential impact: private scripts are run before users creation, so may miss some part of setup.
  * Ensure JupyterHub users exist

A similar set of scripts and features exist for the different components of the solution ([check_api-db.yml](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/check_api-db.yml) for wod-api-db server, [check_frontend.yml ](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/check_frontend.yml) for wod-frontend server for instance).

# Creating a new workshop and integrating it in the WoD platform

Let's start with a simple flowchart describing the 10000-foot view of the creation process:
![WoD creation process](img/wod-creation-process.png "WoD creation process")

As you can see, there's no rocket science here. Just common sense. Depending on the workshop you wish to create, some obvious requirements should show up. A workshop based on a programmatic language, for instance, may require the relevant kernel to be set up on the JupyterHub server. The following [page](https://gist.github.com/chronitis/682c4e0d9f663e85e3d87e97cd7d1624) lists all available kernels. Beware than maintaining many external components such as jupyter kernels may be complicated (been there, done that).

Some workshops might need a specific infrastructure set up in order to run. A Kubernetes 101 workshop, for instance, could not exist without the presence of a proper Kubernetes cluster. You may also have private hardware equipment to manage for your workshop. The private part of the infrastructure is ideal for this use case.

## Workshop: definition and requirements

The [Jupyter Notebook](https://en.wikipedia.org/wiki/Project_Jupyter) is an easy way to transfer knowledge in a way which makes the reader really involved in the execution of the commands he reads. This can be made of multiple notebook JSON files, each containing instructions cells in markdown and run code cells leveraging the relevant kernel (meaning that the code in the cell will be interpreted directly by the underlying code analyzer part of Jupyter, called a kernel). If you are not familiar with Jupyter notebooks, a simple [101 workshop](https://developer.hpe.com/hackshack/workshop/25) is available in HPE's Workshops-on-Demand catalog.
You'll also need to define a student range for the workshop.

A workshop directory should be named `WKSHP-<Name>` (where Name is a free string without space) and should contain at least:

* `0-ReadMeFirst.ipynb` (Name and presence mandatory)
* `1-WKSHP-LAB1.ipynb` (Naming at will linked from the first notebook file)
* `2-WKSHP-LAB2.ipynb` (Naming at will linked from the second notebook file)
* `3-WKSHP-Conclusion.ipynb` (Naming at will linked from the third notebook file)
* `LICENCE.MD` (you don't want your IP to be delivered without license, we recommend [Creative Commons](https://creativecommons.org/share-your-work/cclicenses/))
* `pictures/` (if any screenshot is required in lab instructions use such a folder, name at will)
* `README.md` (content of `0-ReadMeFirst.ipynb` in markdown format, Name mandatory)
* `wod.yml` (Name and presence mandatory)

The 4 first files contains the instructions and code. The first one has to be named `0-ReadMeFirst.ipynb` in order to open automatically in the student's browser. To make the workshop compliant to our platform, you have to provide a set of metadata that will be used for the workshop's integration into the infrastructure (content of a new raw in our database and tile on the wod-frontend). This file is called **`wod.yml`**.

This file is parsed at installation time to feed our database and also during updates operations.
Here is an example of such a file:

```yaml
%YAML 1.1
# Meta data for the GO101 Workshop to populate seeder
---
name: 'GO 101 - A simple introduction to Go Programming Language'
description: 'Go, also called Golang or Go language, is an open source programming language that Google developed. Software developers use Go in an array of operating systems and frameworks to develop web applications, cloud and networking services, and other types of software. This workshop will drive you through the basics of this programming language.'
active: true
capacity: 20
priority: 1
range: [151-170]
reset: false
ldap: false
location: 'fully qualified domain name of default JupyterHub server'
replayId: 31
varpass: false
compile: false
workshopImg: 'https://us-central1-grommet-designer.cloudfunctions.net/images/frederic-passeron-hpe-com/WOD-GO-101-A-simp-introduction-to-Go-programming-language.jpeg'
badgeImg: 'https://us-central1-grommet-designer.cloudfunctions.net/images/frederic-passeron-hpe-com/go101-a-simple-introduction-to-go-programming-language.jpg'
beta: false
category: ['Open Source']
duration: 4
alternateLocation: ['fully qualified domain name of an alternate JupyterHub server']
presenter: 'Matthew Doddler'
role: 'FullStack developer'
avatar: 'img/SpeakerImages/MattD.jpg'
replayLink: 'https://hpe-developer-portal.s3.amazonaws.com/Workshops-on-Demand-Coming-Soon-Replay.mp4'
```

The following fields are required provided by the developer of the notebook:
* **name:** The workshop's name as it will will be displayed on the registration portal
* **description:** The workshop's abstract as it will will be displayed on the registration portal
* **category:** The workshops' registration portal proposes several filters to display the catalog's content. You can view all workshops, the most poular ones, or by category. Use this field to sort workshops accordingly. This should be an array of categories, even if only one is mentionned.
* **duration:** All workshops are time bombed. You will define here the time in hours allocated to perform the workshop
* **workshopImg:** As part of the lifecycle of the workshop, several emails are sent to the student. A workshop image is embedded in them to represent the workshop (optional)
* **badgeImg:** As part of the lifecycle of the workshop, several emails are sent to the student. In the final email, a badge is included. It allows the student to share its accomplishment on social media like linkedin for instance.
* **presenter** Name of the speaker
* **avatar** Image of the speaker in an avatar format
* **role** Role occupied by the speaker in his structure
* **replayLink** This entry links to a dedicated replay video (made earlier) and is displayed when clicking on the learn more page of the workshop
* **replayId** This entry is reserved for the HPE platform at the moment.

**Note:** *Links can be URL pointing to an external server hosting the files, or to the wod-frontend server, under its subdirectory `static/img`*

The following fields are required by the infrastructure and provided by their admins:

* **active:** Tag to set to enable visibility of the workshop's tile in the registration portal and its usage.
* **capacity:** The number of maximum concurrent students allowed to take on the workshop. Linked to what the platform can deliver.
* **range:** The range between which students get picked at registration time. This is a set containing a ,ini,u, and ,axi,u, value. Ranges between workshops should **never** overwrite.
* **reset** entries are to be used by wod-backend server automation if dedicated reset scripts are required by the workshop (cleaning appliances e.g.)
* **ldap** entries are to be used by wod-backend server automation if ldap authentication is required by the workshop
* **location:** If your setup includes multiple JupyterHub servers, use this field to allocate workshops according to your needs. This is a way to load balance the WoD platform.
* **alternateLocation:** If your setup includes multiple JupyterHub servers, use this field to record the other servers where the notebook may be running. This is a way to load balance the WoD platform.
* **compile:** This entry will be filled with the name of scripts to be compiled at deployment time. This feature allows the admin to hide login scripts and credentials in non-editable executable files. The parameter should be a list of coma separated script names without any path surrounded by single quote.
* **varpass:**  This defines whether or not a workshop requires a password variable that needs to be leveraged
* **beta:** Not implemented yet :-)
* **monoAppliance:** Some workshops require a single dedicated appliance
* **multiAppliance:** Some workshops require multiple dedicated appliances

The following file will be used to update the **workshops table** in the database. Let's have a look at what a new entry could look like:
![WoD Go Workshop in DB](img/wod-db-workshop-addition.png "WoD Go Workshop in DB")


## Adding the Workshop to the WoD infrastructure

In this example, we'll add a simple workshop that does not require any infrastructure but the JupyterHub itself. As far as the infrastructure's requirements, a new kernel is needed. No additional scripts are required for this workshop.

As `wodadmin` on the Workshops-on-Demand infrastructure, you have to perform several tasks:

### On the wod-backend server

Testing and validating installation of the new JupyterHub kernel by:

1. Creating a new branch for this test under `wod-backend` (`git checkout -b kernelxyz`).
2. Modifying the [wod-backend server installation yaml file](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/install_backend.yml#L326) to include the new kernel.
3. Push your modifications into your forked repository (`git push myrepo`).
4. Validating the changes by testing a new wod-backend install process from scratch (See upper, remember you can use an `install.priv` file to point to your own branch and your own repository).

Integrate the Jupyter Notebook content from the developer by:

1. Creating a new branch for this test under `wod-notebooks` (`git checkout -b notebookxyz`).
2. Creating the Wokshop directory (`mkdir WKSHP-xyz`)
3. Copying the developer's content into it and ensure the naming conventions exposed upper have been followed. Commit the changes in your branch (`git add -p ; git commit`)
4. Pushing your modifications into your forked repository (`git push myrepo`).
5. Testing the workshop either locally or on the wod-backend server leveraging the `wod-test-action.sh` script.

The live content inside the `wod-notebooks` directory is what will be used during the deployment of the workshop so you can directly use it starting from now.

The developer in charge of building the Jupyter Notebook can use our [existing Notebooks](https://github.com/Workshops-on-Demand/wod-notebooks) as example and use Jupyter locally to make his modifications/development.

Once ready you can make pull requests to the upstream project if you intend to share your modifications with the community or you can keep them private if you prefer. In this case, the best place to host your modifications is under the `wod-private` repository.

### On the wod-api-db server

Integrate the Jupyter Notebook metadata by:

1. Going into the `wod-notebooks` directory and getting the latest content from your repository and branch (`git remote update ; git checkout notebookxyz ; git rebase myrepo`)
2. Move to the `wod-api-db/scripts` directory and update the database by running the `./wod-update-db.sh` script.

This script will update the workshops table inthe PostgreSQL DB with the new workshop entry as well as any modification brought to the various `wod.yml` files from all the workshops.

## Adding a more complex workshop in the infrastructure

In this example, we'll add a comple workshop that does require a specific infrastructure (an appliance, VM or physical server) in addition to the JupyterHub itself, and also additional setup scripts to manage this appliance.

As `wodadmin` on the Workshops-on-Demand infrastructure, you have to perform several tasks:

### On the wod-backend server

Integrate the management of the appliance in the infrastructure by:

1. Updating the relevant variable file to associate the IP address of the server to the workshop (there could be multiple servers too associated to a given workshop) in the YAML `wod-backend/ansible/group_vars/all.yml` file and declare as well potential services ports e.g.
2. Developing a set of scripts under `wod-backend/scripts` or `wod-private/scripts` folders, depending on the nature of the workshop, to manage the workshop's lifecycle
  * the `setup-<WKSHP>.sh` script is in charge of creating the appliance itself, based on a minimal OS install. It could be a container, a VM or a physical machine already installed and on which are performed the final setup steps such as required software installation and configuration.  This script will be called from the `wod-backend/scripts/setup-appliance.sh` during the setup run.
  * the `create-<WKSHP>.sh` script is in charge of instantiate the appliance itself during the workshop deployment. There may be more than one to deploy if your workshop is multiappliance. This script will be called from the `wod-backend/scripts/procmail-action.sh` at deployment time during the CREATE verb management. It should itself call the generic `wod-backend/scripts/create-appliance.sh` to manage ssh keys and perform a preventive cleanup.
  * the `cleanup-<WKSHP>.sh` script is in charge of getting on the appliance any content that was created during the Workshop run before erasure. Typically, the backup of the completion ratio is done in this script. It will be called from the `wod-backend/scripts/procmail-action.sh` during the CLEANUP verb management.
  * the `reset-<WKSHP>.sh` script is in charge of putting the appliance back to its initial state so that idempotency works. Typically, removal of container and container images is done in this script, as well as credential reset and student directory content removed. It will be called from the `wod-backend/scripts/procmail-action.sh` during the RESET verb management. It should also be called at the begining of the `create-<WKSHP>.sh` to be sure we have a clean state before starting. And it should call the generic `wod-backend/scripts/reset-appliance.sh` which resets password and ssh keys created during the CREATE verb.
3. Creating the appliance by running: `wod-backend/scripts/setup-appliance.sh <WKSHP>`. This will:
  * Setup globally all appliances using a common `wod-backend/ansible/setup_appliance.yml` playbook which manages the local Linux users in a similar way than on the wod-backend server.
  * Setup specifically the appliance using a `wod-backend/ansible/setup_<WKSHP>_appliance.yml` playbook which performs the additional configuration steps needed for that type of workshop. For example, installing the Docker engine when running docker images during the workshop.
  * Setup the appliance calling a `wod-backend/script/setup_<WKSHP>.sh` playbook which performs the additional configuration steps needed for that type of workshop. For example, downloading docker images and building new derived ones from them.
4. Testing the workshop using the `wod-test-action.sh` script to test a workshop lifecycle action from deployment (CREATE) to CLEANUP, RESET, or PURGE verbs.

**Note:** *The available trace under `~/.mail/from` will detail the different steps of the action and allow you to troubelshoot any issue.*

When all the scripts are functional and the necessary actions have been performed both on wod-backend and wod-api-db servers, some end to end functional tests can be conducted using the Web interface of the wod-frontend server as described earlier for the simple workshop example.

# Badges

Reward participants with Badges 

![Badges](img/wod-badge-legend.png "Badges")

The [Workshops-on-Demand](https://github.com/Workshops-on-Demand) are a great way to get hands-on experience with the newest and most popular technologies. These Open Source and Free workshops are easy to take and really give you a good
feel for how to interact with tools such as Docker, Ansible, Git, Jupyter, ... We understand that you put in time and effort whenever you take one of our Workshops-on-Demand and we now offer badges in recognition of your achievement.

Now, every time you finish one of our Workshops-on-Demand, you will receive a badge commemorating having completed that specific workshop. You’ll receive the badge in your final, congratulatory email. Share your
badge with friends and colleagues on Mastodon or LinkedIn directly from the email to show them what new skills you’ve recently acquired, typically for Docker:

![Docker Badge](img/wod-badge-Docker101.jpg "Docker Badge")

## More opportunities to collect badges!

But that’s not all! Once you have finished 3 different workshops, you’ll receive an additional badge celebrating your achievement. More badges will be provided once you’ve completed 5, 7, 10, and 15 workshops.
Climb the ranks, from **Apprentice** to **Expert** on to **Hero** and **Super Hero** and, ultimately, to **Legend!** Share these on your social media channels as well, competing with friends and colleagues to see who can make it to the highest level!

![Complete Set of Badges](img/wod-badges-collection.png "Complete Set of Badges")

In addition to any other recognition we grant them, special badges will be awarded to those who create a Jupyter Notebook-based workshop that we can offer to the rest of the WoD Developer Community. Put your thinking caps on to try and collect one of these special badges.
