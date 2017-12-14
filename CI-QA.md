# Review on on CI

## Fuego

<http://fuegotest.org/wiki/Fuego_To_Do_List>

### Fuego Overview

Fuego is a test framework specifically designed for embedded Linux testing.
It supports automated testing of embedded targets from a host system, as it's primary method of test execution.

The quick introduction to Fuego is that it consists of a host/target script engine, with a Jenkins front-end, and over 50 pre-packaged tests, installed in a docker container.

<http://events.linuxfoundation.org/sites/events/files/slides/Introduction-to-Fuego.pdf>

### Install Fuego

<http://fuegotest.org/wiki/Fuego_Quickstart_Guide>

## Kernel CI

kernelci.org is a community based, open source distributed test automation system focused on upstream Linux kernel development. Our goal is to unify all upstream Linux kernel testing efforts in order to provide a single place where to store, view, compare and track these results.

It is our mission to detect, bisect, report and fix regressions on upstream Kernel trees before they even reach «mainline».

## Lava

### LAVA Overview

#### "What is LAVA?"

* LAVA is the Linaro Automation and Validation Architecture.
* LAVA is a continuous integration system for deploying operating systems onto physical and virtual hardware for running tests. Tests can be simple boot testing, bootloader testing and system level testing, although extra hardware may be required for some system tests. Results are tracked over time and data can be exported for further analysis.
* LAVA is a collection of participating components in an evolving architecture. LAVA aims to make systematic, automatic and manual quality control more approachable for projects of all sizes.
* LAVA is designed for validation during development - testing whether the code that engineers are producing “works”, in whatever sense that means. Depending on context, this could be many things, for example:
  * testing whether changes in the Linux kernel compile and boot
  * testing whether the code produced by gcc is smaller or faster
  * testing whether a kernel scheduler change reduces power consumption for a certain workload etc.

* LAVA is good for automated validation. LAVA tests the Linux kernel on a range of supported boards every day. LAVA tests proposed android changes in gerrit before they are landed, and does the same for other projects like gcc. Linaro runs a central validation lab in Cambridge, containing racks full of computers supplied by Linaro members and the necessary infrastucture to control them (servers, serial console servers, network switches etc.)
* LAVA is good for providing developers with the ability to run customised test on a variety of different types of hardware, some of which may be difficult to obtain or integrate. Although * LAVA has support for emulation (based on QEMU), LAVA is best at providing test support for real hardware devices.
* LAVA is principally aimed at testing changes made by developers across multiple hardware platforms to aid portability and encourage multi-platform development. Systems which are already platform independent or which have been optimised for production may not necessarily be able to be tested in LAVA or may provide no overall gain.

#### "What is LAVA not?"

* LAVA is not a set of tests - it is infrastructure to enable users to run their own tests. LAVA concentrates on providing a range of deployment methods and a range of boot methods. Once the login is complete, the test consists of whatever scripts the test writer chooses to execute in that environment.
* LAVA is not a test lab - it is the software that can used in a test lab to control test devices.
* LAVA is not a complete CI system - it is software that can form part of a CI loop. LAVA supports data extraction to make it easier to produce a frontend which is directly relevant to particular groups of developers.
* LAVA is not a build farm - other tools need to be used to prepare binaries which can be passed to the device using LAVA.
* LAVA is not a production test environment for hardware - LAVA is focused on developers and may require changes to the device or the software to enable automation. These changes are often unsuitable for production units. LAVA also expects that most devices will remain available for repeated testing rather than testing the software with a changing set of hardware.

### LAVA-MASTER

The master is a server machine with lava-server installed and it optionally supports one or more remote workers

#### Elements of the Master

* Web interface - This is built using the Apache web server, the uWSGI application server and the Django web framework. It also provides XML-RPC access and the REST API.
* Database - This uses PostgreSQL locally on the master, with no external access.
* Scheduler - This is the piece that causes jobs to be run - periodically this will scan the database to check for queued test jobs and available test devices, starting jobs when the needed resources become available.
* Dispatcher-master daemon - This communicates with the worker(s) using ZMQ.

#### install master

addr=10.20.1.198

add linearo repository to apt source list

```shell
sudo echo "deb https://images.validation.linaro.org/production-repo stretch-backports main" >> /etc/apt/source.list
```

```shell
wget https://images.validation.linaro.org/staging-repo/staging-repo.key.asc
sudo apt-key add staging-repo.key.asc
```

update

```shell
sudo apt upgrade
sudo apt update
```

install lava_server

```shell
sudo apt install postgresql apache2
sudo service postgresql start
sudo apt -t stretch-backports install lava
```

If the default Apache configuration from LAVA is suitable, you can enable it immediately:

```shell
sudo a2dissite 000-default
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2ensite lava-server.conf
sudo service apache2 restart
```

##### Local Django Accounts

After initial package installation, you might wish to create a local superuser account:

```shell
sudo lava-server manage createsuperuser --username $USERNAME --email=$EMAIL
```

If you do not specify the username and email address here, this command will prompt for them.
An existing local Django superuser account can also be converted to an LDAP user account without losing data, using the mergeldapuser command, provided the LDAP username does not already exist in the LAVA instance:

```shell
sudo lava-server manage mergeldapuser --lava-user <lava_user> --ldap-user <ldap_user>
```

### LAVA-worker

The worker is responsible for running the lava-slave daemon to start and monitor test jobs running on the dispatcher. Each master has a worker installed by default. When a dispatcher is added to the master as a separate machine, this worker is a remote worker. The admin decides how many devices to assign to which worker. In large instances, it is common for all devices to be assigned to remote workers to manage the load on the master.

#### Elements of the Worker

* Lava-slave daemon - This receives control messages from the master and sends logs and results back to the master using ZMQ.
* Dispatcher - This manages all the operations on the device under test, according to the job submission and device parameters sent by the master.
* Device Under Test (DUT)

#### Install worker

addr=10.20.1.118

sudo apt --no-install-recommends install python-guestfs

```shell
sudo apt install lava-dispatcher apache2
```

Change the dispatcher configuration in /etc/lava-dispatcher/lava-slave to allow the init script for lava-slave (/etc/init.d/lava-slave) to connect to the relevant lava-master instead of localhost. Change the port numbers, if required, to match those in use on the lava-master:

etc/lava-dispatcher/lava-slave

```config
# Configuration for lava-slave daemon

# URL to the master and the logger
# MASTER_URL="tcp://<lava-master-dns>:5556"
# LOGGER_URL="tcp://<lava-master-dns>:5555"

# Logging level should be uppercase (DEBUG, INFO, WARNING, ERROR)
# LOGLEVEL="DEBUG"

# Encryption
# If set, will activate encryption using the master public and the slave
# private keys
# ENCRYPT="--encrypt"
# MASTER_CERT="--master-cert /etc/lava-dispatcher/certificates.d/<master.key>"
# SLAVE_CERT="--slave-cert /etc/lava-dispatcher/certificates.d/<slave.key_secret>"
```

Restart lava-slave once the changes are complete:

```shell
sudo service lava-slave restart
```

#### Configuring apache2 on a worker

Some test job deployments will require a working Apache2 server to offer deployment files over the network to the device:

```shell
sudo cp /usr/share/lava-dispatcher/apache2/lava-dispatcher.conf /etc/apache2/sites-available/
sudo a2ensite lava-dispatcher
sudo service apache2 restart
wget http://localhost/tmp/
rm index.html
```

You may also need to disable any existing apache2 configuration if this is a default apache2 installation:

```shell
sudo a2dissite 000-default
sudo service apache2 restart
```