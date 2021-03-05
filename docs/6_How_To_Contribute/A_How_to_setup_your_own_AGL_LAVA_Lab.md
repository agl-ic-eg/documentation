
## Prerequisites ##

As well as the packages docker, docker-compose and pyyaml mentioned in the top

level README, you will need the following:

  

1) The following ports are forwarded to docker and therefore need to be kept free

on the host machine:

- 69/UDP: proxyfied to the slave for TFTP

- 80: proxyfied to the slave for TODO (transfer overlay)

- 5500: proxyfied to the slave for Notification

- 55950-56000: proxyfied to the slave for NBD

2) You will need a remote power switch to remotely power the DUTs on and off.

3) You need to have an account with lava.automotivelinux.org. Please contact the

agl-dev-community mailing list if you would like an account, and include that you would

like to create your own lab in the email so that the relevant user permissions

can be set.

  

## Steps to create your own LAVA lab ##

  

1) Clone AGL lava-docker image:

```

git clone https://git.automotivelinux.org/AGL/lava-docker.git

cd lava-docker

```

  

2) On the LAVA master web GUI, create a new API token:

https://lava.automotivelinux.org/api/tokens/

  

3) Connect all the DUTs' serial to usb and ethernet connections to the host.


4) Edit the boards.yaml file:

- Copy the API token you created in step 2 in the place of <generated_lab_token>.

- Add details of each board connected to the lab. See the top level README for

instructions. You will need the following:

- any custom options you require in the kernel args

- uart idvendor, idproduct, devpath

- power on, off and reset commads for the power switch

  

To get the uart idvendor and idproduct, unplug and re-plugin the USB cable of the

device in question and then find the details in the latest output of:

```

sudo dmesg | grep idvendor

```

  

To get the uart devpath, run the command:

```

udevadm info -a -n /dev/ttyUSB1 |grep devpath | head -n1

```

  

NOTE: Make sure you have at least one "board" included. (It is easiest to keep

qemu).

  

5) Run the automated setup script:

```

./start.sh slave

```

  

7) Check the web GUI to see if the lab has successfully connected to the LAVA

master: https://lava.automotivelinux.org/scheduler/allworkers. If it isn't, run the

following command for debugging:

```

docker-exec -it <name_of_docker_container> cat /var/log/lava-dispatcher/lava-slave.log

```

To identify the container name run the following to list the running containers:

```

docker ps

```

  

LAVA logs can be found in `/var/log/lava-dispatcher/`.

  

8) Helper scripts

There are a few helper scripts to automate starting/stopping the lab.

```

./start.sh slave

./restart.sh slave

./stop.sh slave

```

  

## Adding new device-type templates ##

  

Not all device types are supported by default. Templates for new devices will

need to be added to the LAVA master. Please submit new templates to the agl-dev-community

mailing list.

  

Before you submit any new device-type templates, please verify that they work.

You can verify that they work in LAVA by carrying out the following instructions:

1) Install lavacli on Debian Stretch or Ubuntu 18.04 and later (if you don't

have a compatible OS, please see https://lava.automotivelinux.org/api/help/ for an

alternative way to use the API)

2) Create your lavacli config file

```

touch ~/.config/lavacli.yaml

```

3) Configure this file to look like the following (note: use the first token

created in https://lava.automotivelinux.org/api/tokens/)

```

default:

uri: https://lava.automotivelinux.org/RPC2

username: <username>

token: <API_token>

```

4) Add your device template to the master

```

lavacli device-types template set <device-type-name> <device-type-name>.yaml

```

NOTE: make sure your device-type templates always follow the following naming scheme:

```<device-type-name>.yaml```
