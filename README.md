# Cordite Java Demo

Demonstrate how to deploy a CorDapp using Cordite with Java instead of Docker.

The project demonstrates the concept by creating the structure in the build directory.

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
