# smartpan-ami

This project is comprised of a collection of Packer templates and Ansible playbooks which are used in combination to build Amazon Machine Images (AMIs) for a SmartPAN deployment. It is intended to be run inside of a Docker container as part of a CI pipeline (see [infrastructure-packer-runner](https://github.com/companieshouse/infrastructure-packer-runner) for a suitable container).

# Prerequisites

* [Docker](https://www.docker.com/) 2.0.0.3+
* [AWS Command Line Interface](https://aws.amazon.com/cli/) 1.16.155+

## Project hierarchy

The following filesystem structure is used in this project:

```
ansible/
  ├── ...               - Ansible inventory, roles , templates and playbooks
packer/
  ├── vars/             - Packer variable files
  |     └── smartpan    - Variables specific to the 'smartpan' instance
  ├── web.json          - Packer template for building a SmartPAN instance AMI
  └─ ...
```

## Building AMIs

:warning: The recommended way of building AMIs when using this project is to configure a Concourse CI pipeline and run the Packer build process within a Docker container for which there is a purpose-built 'packer' task in [ci-concourse-resources](https://github.com/companieshouse/ci-concourse-resources).

When building AMIs this way, specify the Packer template and variable file to use as part of the build task using the `PACKER_FLAGS` and `PACKER_TEMPLATE_FILE` environment variables respecitely. For full details of the environment variables available refer to the [ci-concourse-resources](https://github.com/companieshouse/ci-concourse-resources) `README` file for the task (i.e. `tasks/ami/packer/README.md`).

For example:

```
  - task: build-ami
    file: concourse-resources/tasks/ami/packer/task.yml
    params:
      ANSIBLE_ROOT: ansible
      PACKER_FLAGS: -var-file=packer/vars/smartpan
      PACKER_TEMPLATE_FILE: packer/smartpan.json
```

It is also possible to build AMIs manually if needed, and this process is described in more detail below.

### Building SmartPAN instance AMIs manually

To manually build a SmartPAN AMI use the AWS command line interface to establish a login to Elastic Container Registry (ECR):

```
$ eval (aws ecr --no-include-email get-login)
Login Succeeded
```

Once successfully logged in, launch a Packer build container with a mount point to this repository (replace `<local-path-to-this-repo>` with the path to the local directory containing this git repository):

```
$ cd <local-path-to-this-repo>
$ docker run --rm -v $(pwd):/source-code -it 123456789012.dkr.ecr.eu-west-1.amazonaws.com/packer-runner /bin/bash
```

This should spawn a container shell with a mount point at `/source-code` containing this repository.

Finally, change directory and export a suitable version number while running the `packer` command and providing the path to both the variable file and Packer template for the desired instance:

```
$ cd /source-code
$ VERSION=1.0.0 packer build -var-file=packer/vars/smartpan templates/smartpan.json
```

