# Continuous Integration and Automated Test (CIAT)

CIAT is the abbreviation of Continuous Integration and Automated Test. It should include:

* CI pipeline which executes tests on user's demand or triggered by upstream changes automatically
* collection of source code from upstream
* automated instructions for building/deploying built upstream components
* ability to include binary artifacts
* automated test pipeline which executes sets of tests
* publication of built distro/component and test results/logs
* mechanism for formal code review prior to merging of changes
* demonstration of license compliance

## Charter

The Continuous Integration and Test Expert Group is responsible for defining requirements and architecture of the AGL software distribution integration and test infrastructure. Topics include:

* Build and smoke test of Gerrit submissions on all hardware
* Daily snapshot build and testing
* Device tests on real hardware
* Test environments such as JTA and Lava
* Test suites such as LTP
* UI testing (OpenQA)

## Current Status

* AGL currently uses Jenkins for CI and Lava + Fuego for running test.
* Jenkins builds for QEMU tied into Gerrit for patch submissions

 The EG is working on these goals:

* Build and smoke test of Gerrit submissions on all hardware
* Daily snapshot build and testing
* Device tests on real hardware
* Integrate test environments such as fuego and Lava
* Investigate UI testing (OpenQA)

## Introduction

AGL uses a combination of tools for its CI tests.

* AGL-JTA (a version of [Fuego](https://elinux.org/Fuego) is the frontend - more information can be found here: [AGL-JTA](https://wiki.automotivelinux.org/agl-jta)
* [Lava](https://validation.linaro.org/static/docs/v2/index.html) as tool to manage the boards remotely.

Both tools can actually work independently, both can execute tests. We combine them to get the best out of both worlds:

* AGL-JTA/Fuego has an extensive predefined set of tests and strong reporting capabilities
* LAVA is capable and excels at managing single boards and board-farms (power-up, deployment of filesystem and so on) and exposes a remote API.

## Use-cases

From the above you can deduct the following use-cases:

* In-house lab - for an isolated, in-house lab, AGL-JTA/Fuego is likely your choice. With a small set of boards on the same network, you only need AGL-JTA/Fuego - no LAVA server is needed.
* Board-Farm - to scale-out the testing and parallelize the execution, we need to manage multiple boards of the same type/family and multiple different boards/brands. In this case you need LAVA to mange the boards.
* Remote Labs - to distribute the boards across sites and manage them, you need LAVA.

## Workflow

The setup supports multiple operation methods:

* Code-change trigger:
  * Gerrit triggers JTA, which runs the test. Board allocation and bring-up is handled by Lava.
* Scripted trigger:
  * Script triggers JTA, which runs the test. Board allocation and bring-up is handled by Lava.
* Manual trigger:
  * Manual trigger in JTA, which runs the test. Board allocation and bring-up is handled by Lava.
* Direct test:
  * Board allocation, bring-up and test execution through Lava.

## Roadmap

Integration of fuego 1.2.1 + LAVA + Jenkins for an industrialisation of build and test

## Jenkins

Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

Jenkins can be installed through native system packages, Docker, or even run standalone by any machine with a Java Runtime Environment (JRE) installed.

### Blue Ocean plugin

Blue Ocean is built as a collection of Jenkins plugins itself. There is one key difference, however. It provides both its own endpoint for http requests and delivers up html/javascript via a different path, without the existing Jenkins UI markup/scripts. React.js and ES6 are used to deliver the javascript components of Blue Ocean. Inspired by this excellent open source project (react-plugins) an <"ExtensionPoint">pattern was established, that allows extensions to come from any Jenkins plugin (only with Javascript) and should they fail to load, have failures isolated.

* Sophisticated visualizations of continuous delivery (CD) Pipelines, allowing for fast and intuitive comprehension of pipeline’s status.
* Pipeline editor makes creation of Pipelines approachable by guiding the user through an intuitive and visual process to create a Pipeline.
* Personalization to suit the role-based needs of each member of the team.
* Pinpoint precision when intervention is needed and/or issues arise. Blue Ocean shows where in the pipeline attention is needed, facilitating exception handling and increasing productivity.
* Native integration for branch and pull requests enables maximum developer productivity when collaborating on code with others in GitHub and Bitbucket.

## Fuego

### Fuego Overview

Fuego is a test framework specifically designed for embedded Linux testing.
It supports automated testing of embedded targets from a host system, as it's primary method of test execution.

The quick introduction to Fuego is that it consists of a host/target script engine, with a Jenkins front-end, and over 50 pre-packaged tests, installed in a docker container.

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

#### Software : LAVA server (master)

* Web nterface
* Job scheduling, priorities
* XML-RPC API
* Board description

  device-type
  * What all boards of this “type” have in common
    * u-boot , fastboot, barebox, etc.
    * Load addresses
    * Bootloader environment
  * Can inherit/extend other device-types (e.g. base-uboot)
  device
  * Specific to one instance of a board
    * Select device-type
    * How to connect to serial console
    * PDU: how to power on/off
    * Can override/extend settings from device-type

### LAVA-worker

The worker is responsible for running the lava-slave daemon to start and monitor test jobs running on the dispatcher. Each master has a worker installed by default. When a dispatcher is added to the master as a separate machine, this worker is a remote worker. The admin decides how many devices to assign to which worker. In large instances, it is common for all devices to be assigned to remote workers to manage the load on the master.

#### Elements of the Worker

* Lava-slave daemon - This receives control messages from the master and sends logs and results back to the master using ZMQ.
* Dispatcher - This manages all the operations on the device under test, according to the job submission and device parameters sent by the master.
* Device Under Test (DUT)

#### Software : LAVA dispatcher (slave) 

##### Manage all connections between boards and “real world”

###### Services

DHCP
TFTP
NFS
NBD
HTTP

###### Serial consoles

USB / serial cables (FTDI)
udev rules
conmux / cu / ser2net

###### USB misc

fastboot
gadget: ethernet, mass storage

## U-boot

(the Universal Bootloader) and Embedded Linux in the form of an editable Wiki.

The "Universal Bootloader" ("Das U-Boot") is a monitor program.
Free Software: full source code under GPL
hosted on SourceForge: <http://sourceforge.net/projects/u-boot>
production quality: used as default boot loader by several board vendors
portable and easy to port and to debug
many supported architectures: PPC, ARM, MIPS, x86, m68k, NIOS, Microblaze
more than 216 boards supported by public source tree
many, many features

easy to port to new architectures, new processors, and new boards
easy to debug: serial console output as soon as possible
features and commands configurable
as small as possible
as reliable as possible

## barebox

barebox is a bootloader designed for embedded systems.
It runs on a variety of architectures including x86, ARM, MIPS, PowerPC and others.

barebox aims to be a versatile and flexible bootloader, not only for booting embedded Linux systems, but also for initial hardware bringup and development.
barebox is highly configurable to be suitable as a full-featured development binary as well as for lean production systems.
Just like busybox is the Swiss Army Knife for embedded Linux, barebox is the Swiss Army Knife for bare metal, hence the name.

## Reference

<http://wiki.kernelci.org/>

<https://github.com/kernelci/lava-docker>

<https://github.com/kernelci/lava-slave-docker>

<http://fuegotest.org/wiki/FrontPage>

<http://fuegotest.org/wiki/Fuego_Quickstart_Guide>

<http://events.linuxfoundationorg/sites/events/files/slides/Introduction-to-Fuego.pdf>

<http://baylibre.com/>

[State of the U-Boot - Thomas Rini](https://www.youtube.com/watch?v=dKBUSMa6oZI)

<http://www.barebox.org/>