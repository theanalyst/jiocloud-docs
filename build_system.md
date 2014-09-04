# Introduction

This document is intended to capture the architecture, implementation, and
usage of JioCloud's automated integration, testing, and deployment frameworks.

## What is CI/CD?

CI/CD ( [continuous integration](http://en.wikipedia.org/wiki/Continuous_integration) and [continuous delivery](http://en.wikipedia.org/wiki/Continuous_delivery) ) are
practices through which changes to code repositories are automatically integrated,
verified, and deployed (also called continuous deployment).

# Architecture Overview

The CI/CD system consumes upstream patches are validates them through
a series of steps in a build pipeline. Each step in the pipeline is intended
to perform a set of testing tasks.

## Guiding Design principals

It's easier to understand many of the design decisions made for this project
within the context of our guiding design principals.

1. Automate everything - Humans make too many mistakes to have them perform
   any manual repetitive tasks. Automating tasks also allows them to scale.

2. No one should ever make any adhoc modification to our cloud systems. This
   leads to risk that certain systems can wind up in an inconsistent state.
   Inconsistent states invalidate our upgrade testing and can lead to.

3. Stay as close to master as possible - The closer you are to master, the
   less code you need to deploy to stay up-to-date. This should limit the
   likelihood that any changeset will lead to a failure.

4. Tests are the gateway for code getting deployed. We have to fully trust
   our automated tests in order to validate that our services are running
   correctly.

5. Build everything expecting it to have to scale massively. Design decisions
   are made assuming that we eventually have to reach 10s or 100s of thousands
   of managed systems. Because of this, we intend to limit the number of
   centralized services as much as possible and expect each of the individual
   servers of our cloud to take on as much of the processing related to their
   own management as possible.

## End to End process:

1. Package build server contains a configuration file that tells it
   what external package and code repositories it should be monitoring
   for changes. When changes are detected, it build out new version of
   the updated packages.

2. Package versioning system periodically (every 15 minutes by default)
   monitors the package build server for updates. When updates are detected,
   it builds a new version (snapshot) that represents the latest version of
   all packages (NOTE: this is related to the time based snapshot of the
   current state of packges which means that it may contain multiple changes))

3. Jenkins monitors the package versioning system for updates. When updates are
   detected, it initializes a new jenkins job that itself sets up a build
   pipeline.

4. The initial job runs what we are calling acceptance tests. These tests
   create a set of virtual machines from scratch and configure them to install
   the desired set of packages. Once the build is verified, tests are run to
   ensure that it results in a functional environment.

5. If this build is successful, it causes more builds to be launched that also
   ensure that servers can be created using the specified snapshot version for:
     * non-functional (performance regression testing)
     * upgrade (verifies that we can upgrade from the
     * staging (verifies that we can provision/upgrade a staging environment
       on bare-metal.
     * production - performs the actual production install/update

## Packages

Package are a central component to our design of CI/CD. All code changes
that will be applied to the pipeline will be stored and configured as packages.

Packages allow us to keep track of all changes, and which specific components
they effect. They also allow us to update only the specific bits of a system
that we care about.

Packages allow us to take advantage of upstream security fixes maintained
by Canonical.

## Puppet

Puppet will be responsible for the configuration of each individual machine into
its desired roles.

For example, if a machine needed to be configured as a database, Puppet is responsible for the logic involved in converting a machine into this role.

### Openstack API

The openstack API will be used for provisioing all virtual machines and
bare-metal servers required for both test environment as well as 


## Installation and Updates

The core infrastructure for running CI/CD is installed and
updated using Puppet and shares as much code and process as possible
with the existing openstack-infra project.

## Developing on CI/CD framework

It is recommended that you test all changes for the CI/CD framework
locally to avoid the likelihood of pushing code that may disrupt the
function of the CI/CD system.

## CI/CD components

### python.jiocloud.apply\_resources

The (python-jiocloud)[https://github.com/jiocloud/python-jiocloud]
project contains python helpers tools that are used as a part
of the build process.

#### Apply Resources

The apply resources code is used to manage stacks of VMs
that need to exist in order for a deployment to match the
desired provisioning specification.

#### Apply Command

The apply command is used to evaluate a specification for which
servers/vms should be provisioned in order to fullfill a specification.

The command is as follows:

````
python -m jiocloud.python apply <resource\_file> <userdata> [--project\tag tag]
````

arguments:

* resource\_file

A file that list the specifications of the stack to be created.

#### Delete Command

The delete command is used to delete specific VMs/servers related to a specific
stack.

###

### Package building system

### Puppet

Puppet is responsible for all data related to the allocation of roles
for nodes or

### Jenkins

Jenkins is responsible for launching all jobs used as a part of the build
pipeline.

## Build workflow
