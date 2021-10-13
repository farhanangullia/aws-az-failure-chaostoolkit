# Chaos Toolkit AZ Failure Extension for AWS

[![Python versions](https://img.shields.io/badge/python-3.6+-blue.svg)](https://www.python.org/downloads/release/python-360/)
[![PyPi version](https://img.shields.io/pypi/v/aws-az-failure-chaostoolkit.svg)](https://pypi.org/project/aws-az-failure-chaostoolkit/#history)
![Tests](https://github.com/awslabs/aws-az-failure-chaostoolkit/workflows/tests/badge.svg?branch=main)
![Lint](https://github.com/awslabs/aws-az-failure-chaostoolkit/workflows/lint/badge.svg?branch=main)
[![Downloads](https://pepy.tech/badge/aws-az-failure-chaostoolkit)](https://pepy.tech/project/aws-az-failure-chaostoolkit)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](./LICENSE)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

> Note: You are strongly advised to only utilize this extension in environments with non-production workloads, as the actions may cause unwanted downtime to your users. Be sure to check if there are any production workloads running in the target AWS account before running Chaos Toolkit experiments with this extension.

This project is a collection of [actions][], gathered as an
extension to the [Chaos Toolkit][chaostoolkit] to simulate an Availability Zone (AZ) failure across multiple AWS services for you to test the resiliency of your hosted applications.

[actions]: https://chaostoolkit.org/reference/api/experiment/#action
[chaostoolkit]: https://chaostoolkit.org

## Install

This package requires Python 3.6 or newer.

To be used from your experiment, this package must be installed in the Python
environment where [chaostoolkit][] already lives.

### Install via pip

```
pip install -U aws-az-failure-chaostoolkit
```

## Usage

To use the actions from this package, add the blocks of code below to your Chaos Toolkit experiment file. Replace `TagKey1` and `TagValue1` with the appropriate key-value pair you tagged your resources with. Replace the value of `az` argument with an availability zone of your choice.

### Failure Actions

#### Auto Scaling Group (ASG)

This action removes subnets belonging to the target AZ in all tagged ASGs and suspends AZRebalance process if its running, or updates the min, max and desired capacity to 0 for the ASG if it's only configured for one AZ:
```yaml
- type: action
  name: Simulate AZ Failure for ASG
  provider:
    type: python
    module: azchaosaws.asg.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      tags:
        - Key: "TagKey1"
          Value: "TagValue1"
```

#### Elastic Compute Cloud (EC2)

This action with network failure will affect tagged/filtered subnets in the target AZ by replacing the current NACL association with a newly created blackhole NACL:
```yaml
- type: action
  name: Simulate AZ Failure for EC2
  provider:
    type: python
    module: azchaosaws.ec2.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      failure_type: "network"
      filters:
        - Name: tag:TagKey1
          Values:
            - "TagValue1"
```

This action with instance failure will affect tagged/filtered instances in the target AZ that are in pending/running state by stopping/terminating normal/spot instances:
```yaml
- type: action
  name: Simulate AZ Failure for EC2
  provider:
    type: python
    module: azchaosaws.ec2.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      failure_type: "instance"
      filters:
        - Name: tag:TagKey1
          Values:
            - "TagValue1"
```

#### Application Load Balancer (ALB)

This action removes subnets from target AZ in tagged application load balancers:
```yaml
- type: action
  name: Simulate AZ Failure for ALB
  provider:
    type: python
    module: azchaosaws.elbv2.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      tags:
        - Key: "TagKey1"
          Value: "TagValue1"
```

#### Classic Load Balancer (CLB)

This action detaches classic load balancers from subnets belonging to target AZ if they are in non-default VPC, and disables target AZ from classic load balancer if they are in a default VPC:
```yaml
- type: action
  name: Simulate AZ Failure for CLB
  provider:
    type: python
    module: azchaosaws.elb.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      tags:
        - Key: "TagKey1"
          Value: "TagValue1"
```

#### Relational Database Service (RDS)

This action forces RDS to reboot and failover to another AZ:
```yaml
- type: action
  name: Simulate AZ Failure for RDS
  provider:
    type: python
    module: azchaosaws.rds.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      tags:
        - Key: "TagKey1"
          Value: "TagValue1"
```

#### ElastiCache

This action forces ElastiCache (cluster mode disabled) to failover primary nodes if exists in the target az:
```yaml
- type: action
  name: Simulate AZ Failure for ElastiCache (cluster mode disabled)
  provider:
    type: python
    module: azchaosaws.elasticache.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      tags:
        - Key: "TagKey1"
          Value: "TagValue1"
```

This action forces ElastiCache (cluster mode enabled) to failover the shards provided as cache cluster ids (sequential if multiple shards of same cluster) (replace ReplicationGroup1, CacheClusterId1 and CacheClusterId2 if needed):
```yaml
- type: action
  name: Simulate AZ Failure for ElastiCache (cluster mode enabled)
  provider:
    type: python
    module: azchaosaws.elasticache.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      tags:
        - Key: "TagKey1"
          Value: "TagValue1"
      replication_groups:
        - replication_group_id: ReplicationGroup1
          cache_cluster_ids:
            - CacheClusterId1
            - CacheClusterId2
```

#### Elastic Kubernetes Service (EKS)

This action removes subnets belonging to the target AZ in all nodegroup ASGs that are part of the tagged EKS clusters and suspends AZRebalance process if its running. Network failure will affect subnets of the nodegroups in the target AZ by associating a newly created blackhole NACL. All its previous NACL association will be replaced with the blackhole NACL:
```yaml
- type: action
  name: Simulate AZ Failure for EKS Clusters
  provider:
    type: python
    module: azchaosaws.eks.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      failure_type: "network"
      tags:
        - TagKey1: "TagValue1"
```

This action removes subnets belonging to the target AZ in all nodegroup ASGs that are part of the tagged EKS clusters and suspends AZRebalance process if its running. Instance failure will affect instances part of the node groups that are in the target AZ that are in pending/running state by stopping normal/spot instances:
```yaml
- type: action
  name: Simulate AZ Failure for EKS Clusters
  provider:
    type: python
    module: azchaosaws.eks.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      failure_type: "instance"
      tags:
        - TagKey1: "TagValue1"
```

#### Managed Message Broker Service (MQ)

This action reboots the specified brokers that are tagged, or tagged brokers if broker_ids not specified:
```yaml
- type: action
  name: Simulate AZ Failure for Amazon MQ (ActiveMQ)
  provider:
    type: python
    module: azchaosaws.mq.actions
    func: fail_az
    arguments:
      az: "ap-southeast-1a"
      dry_run: True
      tags:
        - "TagKey1": "TagValue1"
      broker_ids: 
        - BrokerId1
        - BrokerId2
```

### Tips

* To 'rollback' the changes made by the `fail_az` action, you can use `recover_az` in your experiment template. The `recover_az` function will read the state file generated and rollback if it's a service that's supported.
* Do also note that by default, the `dry_run` argument for each `fail_az` action is required. Setting it to `True` will only run read-only operations and not impact the target resources. Set it to `False` if you want the actions to make changes your resources. It is best practice to set it on an experiment level under the configuration block and then reference it for every action. 
* To have granular filtering of resources, you can also provide a list of tags as part of the argument for the `fail_az` action.

Please explore the code to see existing actions and supported arguments. Alternatively, you can run `chaos discover aws-az-failure-chaostoolkit` to view the list of supported actions along with their required and optional arguments for each service in the generated `discovery.json` file.


## Configuration

### Develop

If you wish to develop on this project, make sure to install the development
dependencies. But first, [create a virtual environment][venv] and then install
those dependencies.

[venv]: http://chaostoolkit.org/reference/usage/install/#create-a-virtual-environment

```console
$ make install-dev
```

Now, you can edit the files and they will be automatically be seen by your
environment, even when running from the `chaos` command locally.

### Format

To format your code execute the following:

```console
$ make fmt
```

### Lint

To check your code with a linter execute the following:

```console
$ make lint
```

### Test

To run the tests for the project execute the following:

```console
$ make test
```

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This project is licensed under the Apache-2.0 License.

