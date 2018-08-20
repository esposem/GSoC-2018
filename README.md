# Qtest Driver Framework
## Introduction
In the current QEMU test implementation, each test suite has to take care of several common steps shared among all the other tests, such as starting QEMU with right devices, allocate the various structures and run the test.
Being open source and having multiple maintainers, each test has basically "invented the weel again", also for very similar devices, like virtio-net-pci and virtio-net-device. 

The aim of this project was to create a framework, qgraph, that is able to give an higher level representation of the tests, introducing semplification in test cases (i.e. not hardcoding a test for a single device of a single architecure, but make it more portable) and improve coverage (same test for virtio-net-pci can be executed for virtio-net-device).

## How it works
Qgraph introduces a different qtest driver
organization, viewing machines, drivers and tests as nodes in a
graph, each having one or multiple edges relations.

The idea is to have a framework where each test asks for a specific
driver, and the framework takes care of allocating the proper devices
required and passing the correct command line arguments to QEMU.

A node can be of four types:
- MACHINE:   for example "arm/raspi2"
- DRIVER:    for example "generic-sdhci"
- INTERFACE: for example "sdhci" (interface for all "-sdhci" drivers)
- TEST:      for example "sdhci-test", consumes an interface and tests
             the functions provided

An edge relation between two nodes (drivers or machines) X and Y can be:
- X CONSUMES Y: Y can be plugged into X
- X PRODUCES Y: X provides the interface Y
- X CONTAINS Y: Y is part of X component

Basic framework steps are the following:
- All nodes and edges are created in their respective machine/driver/test files
- The framework starts QEMU and asks for a list of available devices
  and machines
- The framework walks the graph starting from the available machines and
  performs a Depth First Search for tests
- Once a test is found, the path is walked again and all drivers are
  allocated accordingly and the final interface is passed to the test
- The test is executed
- Unused objects are cleaned and the path discovery is continued

Depending on the QEMU binary used, only some drivers/machines will be available
and only test that are reached by them will be executed.

## Todo
- improve command line creation mechanism, some devices have reference to other command line elemets that are defined in tests (i.e. "e1000e" device command line is `-device e1000e,netdev=hs0,...`, meaning that the final QEMU command line should also have `-netdev socket,id=hs0,...`, otherwise QEMU won't start).
- convert remaining tests

## Repository Overview
This repository contains the directories for the various patchsets that have been sent upstream.

Patch v3-continued (currently under revision):

https://lists.gnu.org/archive/html/qemu-devel/2018-08/msg03901.html

Patch v3 (currently under revision):

https://lists.gnu.org/archive/html/qemu-devel/2018-08/msg02072.html

Patch v2:

https://lists.gnu.org/archive/html/qemu-devel/2018-08/msg00654.html

Patch v1:

https://lists.gnu.org/archive/html/qemu-devel/2018-07/msg02041.html

Others:
- Patch 0001-pci-pc-add-NULL-check-for-qpci_free_pc.patch

https://lists.gnu.org/archive/html/qemu-devel/2018-07/msg04329.html
- Patch 0001-vhost-user-test-added-proper-TestServer-*dest-initialization-in-test_migrate.patch

https://lists.gnu.org/archive/html/qemu-devel/2018-07/msg00221.html
