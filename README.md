# Cordite Java Demo

Demonstrate how to deploy a CorDapp using Cordite with Java instead of Docker.

The project demonstrates the concept by creating the structure in the build directory.

## Build Tasks

There are two extra build tasks added - `deployNetwork` and `copyNMS`.

The normal local deployment bootstraps the local network with files in the `build/nodes` directory .

To remove the bootstrapped files, an extra task `deployNetwork` has been added to create a plain network in `build/network`.

In order to run this plain network you need to have cordite, so the task `copyNMS` will get the jar from Maven Central and put it in the `build/nms` directory.
