# Cordite Java Demo

Demonstrate how to deploy a CorDapp using Cordite with Java instead of Docker.

The Corda samples all use a `deployNodes` task to bootstrap a local network for easy development.

However, in order to run in production you need a network map service. 
R3 provide an NMS as part of Corda Enterprise but this is too expensive for most people.

The open-source alternative is to use the [Cordite NMS](https://gitlab.com/cordite/network-map-service).

This is published as a [docker container](https://hub.docker.com/r/cordite/network-map) but this project will demonstrate how to deploy using Java instead of Docker.

See also the [Cordite FAQ](https://gitlab.com/cordite/network-map-service/-/blob/master/FAQ.md#1-show-me-how-to-set-up-a-simple-network) for further details on setting up a network (both with Docker and Java).

## Create A Network Outline

Create a skeleton outline of local network ready to configure in order to connect to a local instance of cordite.

    ./gradlew deployNetwork
    ./gradlew copyNMS

See below for info about these tasks and how they are different from the usual `deployNodes`.

## Run Cordite

This will start cordite in the following way:

- Store the keys etc in a directory called "storage"
- Create a custom name for the root certificate
- Set the nodes to only check for updates ever 30 seconds

**NOTE:** Setting the admin username/password using command line parameters (`-Dauth-username=AdminUser -Dauth-password=SuperSecretPassword`) is a security risk and they should be set using environment variables instead. For demo purposes only below, we are leaving the default values of sa/admin.

    cd build/nms
    java -jar -Dstorage-type=file -Ddb=storage -Droot-ca-name="CN=Custom Cordite NMS, OU=Operations, O=Demo Company, L=Dallas, ST=Texas, C=US" -Dcache-timeout=30S network-map-service.jar

### Run The Nodes

The following steps are required to start the network:

* Download the truststore from the NMS to each node
* Register all nodes with the NMS
* Start the notary node
* Designate the notary with the NMS
* Stop the notary node
* Delete the network-parameters file on the notary node
* Start the notary node
* Start the other nodes

Get the Network Root Truststore from the NMS and put it in the node directory
    
    cd build/Notary
    mkdir certificates
    cd certificates
    curl -o "network-root-truststore.jks" "http://localhost:8080//network-map/truststore" 

Register the node with the NMS

    cd build/Notary
    java -jar corda.jar initial-registration -p="trustpass"

Start the nodes

    cd build/network/Notary
    java -jar corda.jar

## Build Tasks

There are two extra build tasks added - `deployNetwork` and `copyNMS`.

The normal local deployment bootstraps the local network with files in the `build/nodes` directory .

To remove the bootstrapped files, an extra task `deployNetwork` has been added to create a plain network in `build/network`.

In order to run this plain network you need to have cordite, so the task `copyNMS` will get the jar from Maven Central and put it in the `build/nms` directory.

## Generating Keys

There is a self-signed dev key in the project (`demo-dev-store.pkcs12`) to demonstrate how to sign the contract jar without using the Corda dev keys.

However, you should have proper keys for production.
A production build can sign with the correct keys by passing parameters on the command line.

    ./gradlew jar -Pjar_sign_keystore=/full/path/to/key.pkcs12 -Pjar_sign_alias=AliasName -Pjar_sign_password=Password

The self-signed key in the project was created in the following way:

### Create a private key in JKS format

    keytool -genkey -keyalg RSA -alias demo-dev-key -dname "O=Demo, L=Washington, C=US" -keystore demo-dev-store.jks -storepass password -keypass password

### Convert to PKCS12

The JKS keystore uses a proprietary format, so you need to convert to the industry standard PKCS12 format.

    keytool -importkeystore -srckeystore demo-dev-store.jks -destkeystore demo-dev-store.pkcs12 -deststoretype pkcs12 -srcstorepass password -deststorepass password

After the pkcs12 file is created you no longer need the jks file, so you can just delete it.
