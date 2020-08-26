# Cloud Tools Image - Automated build repository

## Description

This repository contains a Dockerfile that will build a **Ubuntu Bionic** (18.04) image with handy cloud management tools for **Amazon Web Services** and **Oracle Cloud Infrastructure**.

**Important** to know is that this image is generated by a serverless compute unit instance (AWS Lambda function triggered by CloudWatch) whenever a new stable version of **Terraform**, **Packer**, or **Golang Go** is released. **Ansible** version can be set by updating `internal/store.json` file manually by a new commit.

## Software included

Cloud tools and SDKs:
* Terraform **(0.13.1)**
* Packer **(1.6.1)**
* Ansible **(v2.8.0)**
* AWS [CLI](https://aws.amazon.com/cli/) and Python [SDK - boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
* OCI [CLI](https://docs.cloud.oracle.com/iaas/tools/oci-cli/latest/oci_cli_docs/) and Python [SDK](https://oracle-cloud-infrastructure-python-sdk.readthedocs.io/en/latest/)

Development tools:
* build-essential package containing gcc, g++, make **(latest in Bionic)**
* Python **(v3.6)**
* Go **(go1.15)**
* Git, Subversion and some editors like vim, nano, mcedit

Environment:
* Bash with nice prompt coloring
* Bash proxy management function (**proxy**) and terraform OCI environment variables setup function (**setup_terraform_environ**)

**Attention**: Please have a look on the **Dockerfile** in order to see full list of software tools installed.

## Configure the docker container

One good option is to use the container is by creating one image with your non-root user
that you use for Development as follows:

1. Create the container:
```
  docker container run -ti --name <your-container-name> ccurcanu/cloud-tools
```

2. Inside container create your non-root user. You have two options here. Either you create the user with a home folder, or not. In the latter case you'll mount your home into user's home  at container creation.

```
    adduser <your-user-name> --uid <same-like-on-your-machine> # add --no-create-home if required
```

3. Do whatever setup you need inside the container. Outside the container create an image from the container that you're setting up.

```
   docker container ls -a # and see your container id and name <your container name>
   docker container commit <container-id> <new-docker-image-name>
```

4. Start new container and have fun

```
  $ docker container run -ti --name <name> -v /home/<user-folder>:/home/<user-folder> <new-docker-image-name>
  # su your-user # inside the container
```

Notes:

* If you opt for mounting your user home folder into the container user home folder, you need to setup your shell proxy and others by yourself.
* If you don't do the above, you need to create the home folder of the user inside the container and copy the ```.bashrc``` and ```.bashrc_orig``` from /root folder inside it before creating the image.
* You have also the option to use the root user that comes with the container by default, but you'll get in trouble with files generated as root outside the container.


## Configuring shell proxy
For configuring shell proxy you have to create the following files in your **$HOME** and run `proxy` built in command:
* .HTTP_PROXY (one liner file containing something like `http://www.yourproxy.com:80`)
* .HTTPS_PROXY (one liner file containing something like `https://www.yourproxy.com:443`)
* .NO_PROXY  (one liner file containing something like `127.0.0.1,10.0.0.1`)
* .http_proxy (have a look above)
* .https_proxy (have a look above)
* .no_proxy (have a look above)

Beware that the content of those files will be exported to the shell variable having the same name like the file name. So pay attention with spaces after comma, or things that will mess up with shell `export` command.

Proxy command is defined in **$HOME/.bashrc**. `proxy on` will enable shell proxy if above mentioned files are existing. `proxy off` will `unset` the proxy environment variables.

For the lazy people, the following code snippet will help you to easily create the proxy files mentioned above:

```
#!/bin/bash

HTTP_PROXY="<fill-here-your-http-proxy>"
HTTPS_PROXY="<fill-here-your-https-proxy>"
NO_PROXY="<fill-here-the-list-of-no-proxy>"

echo $HTTP_PROXY > $HOME/.http_proxy
echo $HTTP_PROXY > $HOME/.HTTP_PROXY
echo $HTTPS_PROXY > $HOME/.https_proxy
echo $HTTPS_PROXY > $HOME/.HTTPS_PROXY
echo $NO_PROXY > $HOME/.NO_PROXY
echo $NO_PROXY > $HOME/.no_proxy

```

## Configuring AWS access

The easiest way to configure AWS Cloud access is to run [`aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration) which will prompt you for the setup.

Also you can create the configure your **$HOME/.aws/credentials** file as described [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html):

## Configuring OCI access

Configuring OCI console is done in a similar way like the AWS one by running [`oci setup config`](https://docs.cloud.oracle.com/iaas/Content/API/SDKDocs/cliinstall.htm). Variables configured in the **$HOME/.oci/config** must be the following in order for the terraform environment to be successfully configured:
* user (with ocid value)
* tenancy (with ocid value)
* compartment (with ocid value)
* fingerprint
* key_file (with key path)
* region

Based on those the following shell environment variables are exported automatically by shell **setup_terraform_environ** built in function:
* TF_VAR_user_ocid (terraform **user_ocid** variable)
* TF_VAR_tenancy_ocid (terraform **tenancy_ocid** variable)
* TF_VAR_compartment_ocid (terraform **compartment_ocid** variable)
* TF_VAR_fingerprint (terraform **fingerprint** variable)
* TF_VAR_private_key_path (terraform **private_key_path** variable)
* TF_VAR_region (terraform **region** variable)

Dockerfile has version 68.
