# gantryd

Managing Docker-based components across several machines requires a shared view of configuration, container health, and rollout state. gantryd uses Docker and etcd to coordinate those tasks, including **automatic draining** during updates.

## Features

- Store project and component configuration in etcd.
- Run and monitor Docker containers across multiple machines.
- Update components progressively while draining old containers.
- Wait for active connections to finish before shutting down containers.
- Replace unhealthy containers after readiness or health checks fail.
- Manage components locally when required.

## Installation

Install etcd first. Binaries and setup instructions are available from the [etcd releases](https://github.com/coreos/etcd/releases/) and [etcd README](https://github.com/coreos/etcd/blob/master/README.md).

Clone the source and create a Python virtual environment:

```sh
git clone https://github.com/DevTable/gantryd.git
cd gantryd
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

The host also needs Docker and the system packages listed in `requirements.system` or `requirements.system.rhel`, depending on the distribution. For Debian or Ubuntu:

```sh
cat requirements.system | xargs sudo apt-get install -y
```

For RHEL or CentOS:

```sh
cat requirements.system.rhel | xargs sudo yum install -y
```

## Usage

Create a project configuration file. This example defines one Docker component with a port mapping and HTTP checks:

```json
{
  "components": [
    {
      "name": "web",
      "repo": "my/localrepo",
      "tag": "latest",
      "ports": [
        {"external": 8888, "container": 8888}
      ],
      "readyChecks": [
        {"kind": "http", "port": 8888}
      ],
      "healthChecks": [
        {"kind": "http", "port": 8888, "path": "/health"}
      ]
    }
  ]
}
```

Load the configuration into etcd, then update the managed components:

```sh
. .venv/bin/activate
python gantryd.py setconfig project.json
python gantryd.py update
```

Run the commands from a host that can reach the configured etcd service and Docker hosts. The component image, checks, ports, links, volumes, environment variables, and termination behavior are defined in the project configuration.

## License

This project is licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

gantryd is based on [DevTable/gantryd](https://github.com/DevTable/gantryd), which is also distributed under Apache-2.0. See `NOTICE` and the source repository for attribution details.

