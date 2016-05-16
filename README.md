# Cassandra AMI

AMI that should be used to create virtual machines with Cassandra installed
(including clusters).

## Synopsis

This script will create an AMI with Cassandra installed and with all of the
required initialization and configuration scripts.
OpsCenter agent will also be installed on the AMI and configuration scripts
are provided.

## Getting Started

There are a couple of things needed for the script to work.

### Prerequisities

Packer and AWS Command Line Interface tools need to be installed on your
local computer.
To build a base image you have to know the id of the latest Talkdesk Base AMI
files for the region where you wish to build the AMI.

#### Packer

Packer installation instructions can be found [here](https://www.packer.io/docs/installation.html).

#### AWS Command Line Interface

AWS Command Line Interface installation instructions can be found [here](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)

#### Debian AMI's

A list of all the Debian AMI id's can be found at the debian official page:
[Debian oficial Amazon EC2 Images"](https://wiki.debian.org/Cloud/AmazonEC2Image/)

### Usage

In order to create the AMI using this packer template you need to provide a
few options.

```
Usage:
  packer build -var 'aws_access_key=AWS_ACCESS_KEY' -var 'aws_secret_key=<AWS_SECRET_KEY>' -var 'aws_region=<AWS_REGION>' -var 'aws_base_ami=<BASE_IMAGE>' -var 'cassandra_version=<CASSANDRA_VERSION>' -var 'java_version=<JAVA_VERSION>' -var 'opscenter_version=<OPSCENTER_VERSION>' cassandra.json
Options:
  aws_access_key         the aws access key for your user.
  aws_secret_key         the aws secret key for your user.
  aws_region             the region where the ami will be created.
  aws_base_ami           the debian image id to use for the build.
  cassandra_version   the cassandra version to install.
  java_version        the java version used by cassandra (will be installed).
  opscenter_version   the opscenter agent version to install.
```

### Instantiate a Server

In order to end up with a functional Cassandra server some configurations have
to be performed after instantiating a new server.

#### Configuring a Cassandra node

To prepare an instance to act as a Cassandra node the following steps need to
be performed.

Run the configuration tool (cassandra_config) to configure the Cassandra node.

```
Usage: cassandra_config [-a -d -e -M <HEAP_NEWSIZE> -m <HEAP_MAXSIZE> -S (-s <SEED_ADDR>|...) -v <NUM_TOKENS>] -n <CLUSTER_NAME>
Options:
  -a                      Enable auto bootstrap.
  -d                      Disable starting service at boot.
  -D                      Delay the service startup by 120s (if -S is used).
  -e                      Enable starting service at boot.
  -m <HEAP_MAXSIZE>       The amount of memory to be allocated to the head size (e.g.: 2G).
  -M <HEAP_NEWSIZE>       The amount of memory to be allocated to the head new size (e.g.: 200M).
  -n <CLUSTER_NAME>       The custer name.
  -R                      Change to the EC2MultiRegionSnitch.
  -s <SEED_ADDR>          The address of a seed server.
  -S                      Start the service after configuring it.
  -v <NUM_TOKENS>         The number of vnodes to use.
```

In a production environment it is recommended to enable the service to start
at boot (-e) so that the service can start on a server restart.

Here is an example of how the configuration command can be executed:

```
cassandra_config -e -m 1G -M 100M -s 10.0.20.1 -s 10.0.20.2 -S -D -v 32 -n "your cluster"
```

After this steps a Cassandra node should be configured and (if chosen) running
on the server.

#### Configuring the OpsCenter Agent

To prepare the included OpsCenter agent the following steps need to be
performed.

Run the configuration tool (opscenter_agent_config) to configure the OpsCenter
agent.

```
Usage: opscenter_agent_config [-d -e -S ] -s <OPSCENTER_ADDR>
Options:
  -d                      Disable starting service at boot.
  -e                      Enable starting service at boot.
  -s <OPSCENTER_ADDR>     The address of the opscenter server.
  -S                      Start the service after configuring it.
```

In a production environment it is recommended to enable the service to start
at boot (-e) so that the service can start on a server restart.

Here is an example of how the configuration command can be executed:

```
opscenter_agent_config -e -s opscenter.yourdomain.com -S
```

After this steps the OpsCenter agent should be configured and (if chosen)
running on the server.

## Services

This AMI will have the SSH service running as well as the Cassandra Node. The
following ports will have to be configured on Security Groups.

| Service   | Port   | Protocol |
|-----------|:------:|:--------:|
| ssh       |   22   |    TCP   |
| cassandra |  7000  |    TCP   |
| cassandra |  7001  |    TCP   |
| cassandra |  7199  |    TCP   |
| cassandra |  9042  |    TCP   |
| cassandra |  9160  |    TCP   |

Details on the Cassandra ports can be found [here](https://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureFireWall_r.html)

## Contributing

1. Create your feature branch: `git checkout -b my-new-feature`
2. Commit your changes: `git commit -am 'Add some feature'`
3. Push to the branch: `git push origin my-new-feature`
4. Submit a pull request

## Versioning

This project uses [SemVer](http://semver.org/) for versioning. For the versions
available, see the [tags on this repository](https://github.com/fscm/packer-templates/tags).

## Authors

* **Frederico Martins** - [fscm](https://github.com/fscm)

See also the list of [contributors](https://github.com/fscm/packer-templates/contributors)
who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/fscm/packer-templates/blob/master/LICENSE)
file for details
