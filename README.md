# Apache Cassandra AMI

AMI that should be used to create virtual machines with Apache Cassandra
installed.

## Synopsis

This script will create an AMI with Apache Cassandra installed and with all of
the required initialization scripts.

The AMI resulting from this script should be the one used to instantiate a
Cassandra server (standalone or cluster).

## Getting Started

There are a couple of things needed for the script to work.

### Prerequisites

Packer and AWS Command Line Interface tools need to be installed on your local
computer.
To build a base image you have to know the id of the latest Debian AMI files
for the region where you wish to build the AMI.

#### Packer

Packer installation instructions can be found
[here](https://www.packer.io/docs/installation.html).

#### AWS Command Line Interface

AWS Command Line Interface installation instructions can be found [here](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)

#### Debian AMI's

A list of all the Debian AMI id's can be found at the debian official page:
[Debian oficial Amazon EC2 Images](https://wiki.debian.org/Cloud/AmazonEC2Image/)

### Usage

In order to create the AMI using this packer template you need to provide a
few options.

```
Usage:
  packer build \
    -var 'aws_access_key=AWS_ACCESS_KEY' \
    -var 'aws_secret_key=<AWS_SECRET_KEY>' \
    -var 'aws_region=<AWS_REGION>' \
    -var 'aws_base_ami=<BASE_IMAGE>' \
    -var 'cassandra_version=<SPARK_VERSION>' \
    [-var 'option=value'] \
    cassandra.json
```
#### Script Options

- `aws_access_key` - *[required]* The AWS access key.
- `aws_ami_name` - The AMI name (default value: "cassandra").
- `aws_ami_name_prefix` - Prefix for the AMI name (default value: "").
- `aws_base_ami` - *[required]* The AWS base AMI id to use. See [here](https://wiki.debian.org/Cloud/AmazonEC2Image/) for a list of available options.
- `aws_instance_type` - The instance type to use for the build (default value: "t2.micro").
- `aws_region` - *[required]* The regions were the build will be performed.
- `aws_secret_key` - *[required]* The AWS secret key.
- `cassandra_version` - *[required]* Cassandra version.
- `java_build_number` - Java build number (default value: "15").
- `java_major_version` - Java major version (default value: "8").
- `java_update_version` - Java update version (default value: "112").
- `system_locale` - Locale for the system (default value: "en_US").

### Instantiate a Cluster

In order to end up with a functional Cassandra Cluster some configurations have
to be performed after instantiating the servers.

To help perform those configurations a small script is included on the AWS
image. The script is called **cassandra_config**.

#### Configuration Script

The script can and should be used to set some of the Cassandra options as well
as setting the Cassandra service to start at boot.

```
Usage: cassandra_config [options]
```

##### Options

* `-a` - Sets the auto_bootstrap option to 'on'.
* `-c <NAME>` - Sets the cluster_name option to the value given (default value is 'Cassandra Cluster').
* `-D` - Disables the Cassandra service from start at boot time.
* `-E` - Enables the Cassandra service to start at boot time.
* `-i <ADDRESS>` - Sets the cassandra listen address (default value is '127.0.0.1').
* `-m <MEMORY>` - Sets Cassandra maximum heap size. Values should be provided following the same Java heap nomenclature.
* `-n <MEMORY>` - Sets Cassandra heap new size. Values should be provided following the same Java heap nomenclature.
* `-r` - Sets the endpoint_snitch option to 'EC2MultiRegionSnitch' (default value is 'SimpleSnitch').
* `-s <ADDRESS>` - Sets the address of one seed node (or several seed nodes if the addressess are separated with a comma).
* `-S` - Starts the Cassandra service after performing the required configurations (if any given).
* `-t <NUMBER>` - Sets the number of tokens (default value is '4').
* `-W <SECONDS>` - Waits the specified amount of seconds before starting the Cassandra service (default value is '0').

#### Configuring a Cassandra Node

To prepare an instance to act as a Cassandra cluster node the following steps
need to be performed.

Run the configuration tool (*cassandra_config*) to configure the instance.

```
cassandra_config -E -S
```

After this steps a Cassandra node (for a single node cluster) should be running
and configured to start on server boot.

For a cluster with more than one Cassandra node other options have to be
configured on each instance using the same configuration tool
(*cassandra_config*).

```
cassandra_config -c 'My Cluster' -E -i 10.0.0.1 -s 10.0.0.1 -s 10.0.0.2,10.0.0.3 -S
```

After this steps, the first node of the Cassandra cluster (for a three node
cluster) should be running and configured to start on server boot.

More options can be used on the instance configuration, see the
[Configuration Script](#configuration-script) section for more details

## Services

This AMI will have the SSH service running as well as the Cassandra services.
The following ports will have to be configured on Security Groups.

| Service                            | Port   | Protocol |
|------------------------------------|:------:|:--------:|
| SSH                                | 22     |    TCP   |
| Cassandra Inter-Node Cluster       | 7000   |    TCP   |
| Cassandra SSL inter-node cluster   | 7001   |    TCP   |
| JMX Monitoring                     | 7199   |    TCP   |
| Cassandra CQL Native Transport     | 9042   |    TCP   |
| Cassandra SSL CQL Native Transport | 9142   |    TCP   |
| Cassandra Thrift                   | 9160   |    TCP   |

More details can be found
[here](http://docs.datastax.com/en/cassandra/3.x/cassandra/configuration/secureFireWall.html?hl=firewall).

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request

## Versioning

This project uses [SemVer](http://semver.org/) for versioning. For the versions
available, see the [tags on this repository](https://github.com/fscm/packer-templates/tags).

## Authors

* **Frederico Martins** - [fscm](https://github.com/fscm)

See also the list of [contributors](https://github.com/fscm/packer-templates/contributors)
who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/fscm/packer-templates/LICENSE)
file for details
