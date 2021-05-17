# DevOps tool

# Build Automation Tools

>> build automation is automated processing of code in preparation for deployment

tools depend on programing language, for example

1. java → ant, maven, gradle
2. javascript → npm, grunt, gulp
3. make → widely used in Unix-based system
4. packer → build machine image and container

# Continuous Integration Tools

>> continuous integration is continuously merging code into a single branch or mainline

tools usually consist of a server that integrates with source control → source control trigger event to CI tools → rework

1. Jenkins → dashboard
2. Travis CI → github & execute in clean VMs
3. Bamboo → not free & 

# Configuration Management Tools

>> configuration management is managing and changing the state of pieces of infrastructure in a consistent and maintainable way → implement "infrastructure as code"

1. Ansible → specific desire state (declarative configuration) & no require control server (but also have ansible tower) & less requirement (only python, ssh) & yaml file
2. Puppet → desire state & dashboard & require control server & uses Puppet DSL (domain specific language)
3. Chef  → tell how to go to desire state (procedural configuration) & require control server & uses Chef DSL (domain specific language)
4. Salt → desire state & require control server & yaml file & support event-driven automation (other tools require human)

# Virtualization

>> virtualization is managing resources by creating virtual rather physical machine → virtual machine (VMs)

# Containerization

>> containers is light weight, isolated packages containing everything needed to run a piece of software

# Monitoring Tools

>> monitoring is collecting and presenting data about the state and performance of application, resource usage

## Infrastructure Monitoring Tools

>> focus on thing relate to infra.

example → CPU, ram

1. SenSu → design for Nagios & server/agent
2. NewRelic -> SaaS & server/agent & many metrics

## Application Performance Monitoring Tools

>> focus on performance and stability of individual parts of an application

1. AppDynamics → code level problem detection &  serer/agent    
2. NewRelic

## Aggregation and Analytics Tools

>> are about collecting monitoring data and doing something with it

1. Elastic Stack → diagnose problems

# Orchestrations Tools

>> automation that supports processes and workflows

- scale up and down application on request
- auto scale
- desire state
1. Docker Swarm
2. Kubernetes
3. Zookeeper
4. Terraform → combine orchestration and infrastructure as code & AWS & Kubernetes & Ansible