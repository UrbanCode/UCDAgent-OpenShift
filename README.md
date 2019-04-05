# UrbanCode Deploy Agent - OpenShift Template

[UrbanCode Deploy Agent](https://developer.ibm.com/urbancode/products/urbancode-deploy/) is a tool for automating application deployments through your environments. It is designed to facilitate rapid feedback and continuous delivery in agile development while providing the audit trails, versioning and approvals needed in production.

## Introduction

This chart deploys a single instance of the UrbanCode Deploy agent.

## Template Details
* Includes a StatefulSet workload object

## Prerequisites

1. The agent must have a UrbanCode Deploy server or relay to connect to.

2. A PersistentVolume that will hold the conf directory for the UrbanCode Deploy agent is required.  If your cluster supports dynamic volume provisioning you will not need to create a PersistentVolume (PV) or PersistentVolumeClaim (PVC) before installing this template.  If your cluster does not support dynamic volume provisioning, you will need to either ensure a PV is available or you will need to create one before installing this template.  You can optionally create the PVC to bind it to a specific PV, or you can let the chart create a PVC and bind to any available PV that meets the required size and storage class.  Sample YAML to create the PV and PVC are provided below.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ucda-conf-vol
  labels:
    volume: ucda-conf-vol
spec:
  capacity:
    storage: 10Mi
  accessModes:
    - ReadWriteOnce
  nfs:
    server: 192.168.1.17
    path: /volume1/k8/ucda-conf
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ucda-conf-volc
spec:
  storageClassName: ""
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: 10Mi
  selector:
    matchLabels:
      volume: ucda-conf-vol
```

## Resources Required
Kubernetes 1.9

## Application deployment using the template

Use the following command to deploy the UCD Server to an OpenShift project:

```bash
$ oc process -f https://raw.githubusercontent.com/IBM-UrbanCode/UCDAgent-OpenShift/master/ucda_template.yaml --param-file values.txt | oc create -f -
```
The above command uses the values.txt file to supply parameter values for the application deployment.
Here is the example contents of a values.txt file:
```
APPLICATION_NAME=my-ucd-agent
IMAGE_REPOSITORY=mydocker_repo/prod/ucda
IMAGE_TAG=7.0.2.2.1017795
IMAGE_PULL_POLICY=Always
IMAGE_PULL_SECRET=my_repository_secret
SERVER_URI=wss://10.134.119.240:32537
```

The [configuration](#Configuration) section lists the parameters that can be set during installation.


## Configuration

### Parameters

The template uses the following parameter values that can be supplied in a separate parameters file.

##### Common Parameters

| Qualifier | Parameter  | Definition | Allowed Value |
|---|---|---|---|
| application name |  | The name of the deployed application | |
| image | pullPolicy | Image Pull Policy | Always, Never, or IfNotPresent. Defaults to Always if :latest tag is specified, or IfNotPresent otherwise  |
|       | repository | Name of image, including repository prefix (if required) | See [Extended description of Docker tags](https://docs.docker.com/engine/reference/commandline/tag/#extended-description) |
|       | tag | Docker image tag | See [Docker tag description](https://docs.docker.com/engine/reference/commandline/tag/) |
|       | secret |  An image pull secret used to authenticate with the image registry | Empty (default) if no authentication is required to access the image registry. |
| persistence | enabled | Determines if persistent storage will be used to hold the UCD server appdata directory contents. This should always be true to preserve server data on container restarts. | Default value "true" |
|             | useDynamicProvisioning | Set to "true" if the cluster supports dynamic storage provisoning | Default value "false" |
| confVolume | name | The base name used when the Persistent Volume and/or Persistent Volume Claim for the UCD agent conf directory is created by the chart. | Default value is "conf" |
|               | existingClaimName | The name of an existing Persistent Volume Claim that references the Persistent Volume that will be used to hold the UCD agent conf directory. |  |
|               | storageClassName | The name of the storage class to use when persistence.useDynamicProvisioning is set to "true". |  |
|               | size | Size of the volume to hold the UCD agent conf directory |  |
| relayUri |  | Agent relay URI if the agent is connecting to a relay. If multiple relays are specified, separate them with commas. For example, random:(http://relay1:20080,http://relay2:20080) |  |
| serverUri |  | UCD server URI. If multiple servers are specified, separate them with commas. For example, random:(wss://ucd1.example.com:7919,wss://ucd2.example.com:7919) |  |
| agentTeams |  | Teams to add this agent to when it connects to the UCD server.Format is <team>:<type>. Multiple team specifications are separated with a comma. |  |

## Storage
See the Prerequisites section of this page for storage information.

## Limitations


