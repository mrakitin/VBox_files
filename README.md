This is a short README file regarding the VirtualBox shared installation on
**nsls2expdev1** Windows 2012 server.

I. VirtualBox shared directory
--
By default, each user has his own configuration & machines in the VirtualBox 
environment. As of today, VirtualBox was configured to use a shared directory
available to all users, which is at `D:\VMs\`.

The instructions for multiple users access are adapted from
http://lifeofageekadmin.com/allow-multiple-users/:
- `VBOXUSERS` group was created with 2 users (`BNL\chubar` and `BNL\mrakitin`);
- `VBOX_USER_HOME` environment variable was set to `D:\VMs\.VirtualBox` (that's
  the path where all configuration files of the VirtualBox installation are
  stored.

II. VirtualBox service
--
For MS Windows, VirtualBox does not have an option to run a VM as a service. For
that purpose third-party software packages exist. I've choosen a free 
implementation [VBoxVmService](http://vboxvmservice.sourceforge.net/). This
allows to install a service for a VM to run as the impersonated user (does not
require to run a service as a domain user => no need to change the password for
the service every 3-6 months). The instructions for configuration of the service
are available at http://techgenix.com/start-virtualbox-service/. The package is
installed in `D:\VMs\VBoxVmService\`, the [VBoxVmService.ini](VBoxVmService.ini) file should look like:
```
[Settings]
VBOX_USER_HOME=D:\VMs\.VirtualBox
RunWebService=no
PauseShutdown=30000

[Vm0]
VmName=Sirepo
ShutdownMethod=savestate
AutoStart=yes
```

The service was installed in the Administrator mode by the following commands:
```bat
> D:\
> cd D:\VMs\VBoxVmService\
> VmServiceControl.exe -i
```
or use `VmServiceControl.exe -u` and then `VmServiceControl.exe -i` to reinstall the service.

The startup type of the created service `VBoxVmService` should be updated to be 
`Automatic (Delayed Start)` to run 2 minutes after all Windows services have
started.

**Note:** the GUI will not display correct status of the VM since it runs in a
separate environment. Check the logs at `D:\VMs\VBoxVmService\VBoxVmService.log` 
for the correct status information. Something like the following should be in
the logs:
```
09/05/2017, 10:05:46 - VBoxVmService started.
09/05/2017, 10:05:47 - List all the VMs found by VBoxVmService
09/05/2017, 10:05:47 -   VM0: Sirepo is saved.
09/05/2017, 10:05:53 -   VM Sirepo has been started up.
```

The structure of `D:\VMs\`:
- `.VirtualBox\`    - general VirtualBox configuration files
- `VBoxVmService\`  - VBoxVmService files
- `VirtualBox VMs\` - Sirepo VM configuration and disk files

III. Sirepo services inside Sirepo VM:
--
Connect over ssh to `nsls2expdev1.bnl.gov:2222` with vagrant user and run:
```
screen -R -D my
```

![](img/sirepo_screen.png)

To switch between screens use <kbd>Ctrl></kbd>+<kbd>^</kbd> then <kbd>Space</kbd> bar.

- screen 0:
```bash
htop
```

- screen 1:

If there is an existing container, clean it by the [command](https://github.com/mrakitin/utils/blob/master/bash/clean_docker.sh):
```bash
bash ~/src/mrakitin/utils/bash/clean_docker.sh mgmt
```
Then run:
```bash
docker pull rabbitmq:management
docker run --rm --hostname rabbit --name rabbit -p 5672:5672 -p 15672:15672 rabbitmq:management
```

- screen 2:
```bash
celery worker -A sirepo.celery_tasks -l info -Q parallel,sequential
```

- screen 3:
```bash
celery flower -A sirepo.celery_tasks
```

- screen 4:
```bash
sirepo service http
```

- screen 5 (used for console operations):
```bash
cd ~/src/radiasoft/sirepo
git pull --all  # pull the latest version of Sirepo
```

IV. Troubleshooting
--
If Sirepo fails to open any of the predefined examples (e.g., Undulator Radiation), it may mean the server is missing some of the simulaiton codes.

To install the missing packages, execute the following commands:
```bash
cd ~/src/radiasoft/sirepo
pip install -r requirements.txt
```

To install WARP, do the following:
```bash
curl radia.run | bash -s code warp rsbeams
```

---
2017-09-06 by Maksim Rakitin
