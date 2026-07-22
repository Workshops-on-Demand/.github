![hpedevlogo2](https://github.com/Workshops-on-Demand/.github/assets/25387895/05d10b28-4447-4b94-8169-f79e617ccde7)

![WoD frontend](img/wod-frontend.png)

# Workshops-on-Demand architecture

What is the goal of the Workshops-on-Demand (WoD) ? Make it easy to register for and run jupyter notebooks you can host yourself thanks to this infrastructure.

![100 feet view of the WoD Architecture](img/wod-howto.png)

The WoD infrastructure comprises 3 differents systems to work, that are usually spread across 3 machines:
 
* a wod-backend machine, hosting the Jupyter Hub and the WoD templates to generate the real workshop that a given student will run, with their metadata. This machine may also interact with appliances for WoD needing one, such as Docker e.g. You may have multiple wod-backend in case of a large setup. Corresponding software on the repos [wod-backend](https://github.com/Workshops-on-Demand/wod-backend), [wod-notebooks templates](https://github.com/Workshops-on-Demand/wod-notebooks), [wod-private optional setup](https://github.com/Workshops-on-Demand/wod-private)
* a wod-api-db machine hosting the WoD API service and a PostgreSQL database to store live information about the running platform. Corresponding software on the repo [wod-api-db](https://github.com/Workshops-on-Demand/wod-api-db)
* a wod-fronted machine, hosting the Web interface to see the list of available workshops and book one. Corresponding software on the repo [wod-frontend](https://github.com/Workshops-on-Demand/wod-frontend)

All systems are installed using our [wod-install](https://github.com/Workshops-on-Demand/wod-install) installer.

It is built on shoulders of giants
![Tools used by the WoD infrastructure](img/wod-repositories.png)

The WoD architecture uses REST and MAIL API to allow communication between its nodes
![REST and MAIL API communications](img/wod-infra-open-source.png)

Before a note book is made available to the user requesting it, there are a certain number of communications happening between machines
![Ordered communications](img/wod-protocols.png)

* 1/ The user chooses the Workshop to run on the WoD Frontend portal through an https request
* 2/ The WoD Frontend exchanges with the WoD API-DB server to setup the customer, and manage the request of workshop (availability, capacity, ...). The infos generated are stored in the DB.
* 3/ The WoD API-DB server sends a mail to the procmail service on the WoD backend in order to setup the required elements (jupyter notebook, potentially appliances, resources, ... in 3b). The full deployment is managed with shell scripts and ansible playbooks.
* 4/ The WoD API-DB server prepares e-mails to be sent by the WoD Backend server to the user to give appropriate welcome and credentials.
* 5/ The user runs his Jupyter notebook Workshop on Demand by connecting through https to the WoD Backend Jupyter Hub.
* 6/ The Notebook may access additional resources managed by appliances (storage, specific applications, Hardware components, ...)
* 7/ The WoD Backend server manages status and progresses by making https REST API calls to the WoD API-DB server, up to the end of the time slot.
* 8/ The WoD API-DB server keeps track of live data concerning the user and the workshops in its DB.
* 9/ The WoD Backend server warns the user of time remaining up to the end, and sends also polls when the Workshop on Demand is finished.
* 10/ The user fills the poll ans wins badges.


# Workshops-on-Demand documentation

First you may want to read the [User's guide](USER-GUIDE.md) to understand how to use the WoD solution, and how it's working from a user's perspective.

Then there is the [technical documentation](ADMIN-GUIDE.md) describing the genesis and the architecture of the project. It also covers the deloyment phase of the different components as well as the creation of content.

These articles were originally published on the [HPE Developer Community Portal](https://developer.hpe.com/blog) as blog articles.

* 0/ [From Jupyter Notebooks-as-a-Service to HPE DEV Workshops-on-Demand](https://developer.hpe.com/blog/from-jupyter-notebooks-as-a-service-to-hpe-dev-workshops-on-demand/)
* 1/ [Open sourcing Workshops-on-Demand part 1: Why and How](https://developer.hpe.com/blog/willing-to-build-up-your-own-workshops-on-demand-infrastructure/)
* 2/ [Open Sourcing Workshops-on-Demand part 2: How to Deploy the infrastructure](https://developer.hpe.com/blog/open-sourcing-workshops-on-demand-part-2-how-to-deploy-the-infrastructure/)
* 3/ [Open Sourcing Workshops-on-Demand part 3: Understand the Backend](https://developer.hpe.com/blog/open-sourcing-workshops-on-demand-part3-understanding-the-backend/)
* 4/ [Open Sourcing Workshops-on-Demand part 4: Manage the Backend](https://developer.hpe.com/blog/open-sourcing-workshops-on-demand-part4-managing-the-backend/)
* 5/ [Open Sourcing Workshops-on-Demand part 5: Create new workshops](https://developer.hpe.com/blog/open-sourcing-workshops-on-demand-part-5-create-a-workshop/)

You can also refer to the various presentations made on this project:
* [LCA 2021](https://www.youtube.com/watch?v=D6Ss3T2p008)
* [OSXP 2021](https://www.slideshare.net/HuinLucile/api-rest-procmail-la-rescousse) and the [Video](https://www.youtube.com/watch?v=zZm6ObQATDI)
* [JDLL 2024](https://pretalx.jdll.org/jdll2024/talk/VUEQFM/)
* [AlpOSS 2024](https://alposs.fr/2024/presentations/03%20-%20HPE%20-%20Bruno%20Cornec.pdf)
* [HPE Developer Community Meetup 2025](https://www.youtube.com/watch?v=RSt7ewTulPE)

# Installing a WoD infrastructure

In order to install a full infrastructure, a set of reliminary steps are required:

* Install a VM/Physical machine with [Ubuntu 24.04 LTS](https://www.ubuntu-fr.org/download/) minimal and default setup. (20.04 or 22.04 should still work, while less tested these days).
* On each VM the user ubuntu is being created and can be used for the initial setup. For access to the machine and account refer to the [Ubuntu documentation](https://ubuntu.com/server/docs/basic-installation) or how a VM template was set up. Ensure that this user as root access (either via sudo or with password access)
* Then you have to ensure minimal dependencies are present to run the installation. We only need git. All other dependencies are installed by the installer. So issue on each machine: `sudo apt install git`
* Once installed you can use it to clone the installer (part of the [install project](https://github.com/Workshops-on-Demand/wod-install)) again on each machine: `git clone https://github.com/Workshops-on-Demand/wod-install`
* Then you use the installer to install your WoD infrastructure: `cd wod-install/install ; ./install.sh -h`
* [Reading the example](https://github.com/Workshops-on-Demand/wod-install/blob/main/install/install.sh#L89) for the full infrastructure at the end of the help message should give you the required guidance to set it up.

At the time of this writing, this reads:
```
Full installation example of a stack with:
- 2 backend servers be1 and be2 using port 8010
- 1 api-db server apidb on port 10000
- 1 frontend server front on port 8000
- all declared on the .local network
- internal postfix server running on port 9000
- e-mail sender being wodmailer@local
- ansible groupname being test
- management user being wodmgr
 
On the be1 machine:

  sudo ./install.sh -a apidb.local:10000:https -f front.local:8000:https \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -b be1.local:8010 -n 1 -t backend \
On the be2 machine:
  sudo ./install.sh -a apidb.local:10000:https -f front.local:8000:https \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -b be2.local:8010 -n 2 -t backend \
On the apidb machine:
  sudo ./install.sh -a apidb.local:10000:https -f front.local:8000:https \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -b be1.local:8010,be2.local:8010 -t api-db \
On the frontend machine:
  sudo ./install.sh -a apidb.local:10000:https -f front.local:8000:https \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -t frontend \
```

Note that the installer should support Rocky/Alma or Debian fairly easily instead of Ubuntu, but tests have not been fully performed. Contributions welcome.
