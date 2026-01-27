# Carbonio Erlang

This repository contains packaging configurations for Erlang and Elixir runtime environments, specifically tailored for the Carbonio platform. It includes:

- **carbonio-erlang**: A headless version of Erlang/OTP compiled with SMP support, built-in zlib, ODBC, and SSL support
- **carbonio-elixir**: Elixir compiled to work with the packaged Erlang runtime

These packages are built for both DEB and RPM-based Linux distributions and are optimized for use within the Carbonio ecosystem.

## Quick Start

### Prerequisites

- Docker or Podman installed
- Make

### Building Packages

This project requires third-party dependencies (like `carbonio-openssl`) at build time.

#### For Zextras Developers (with Artifactory access)

Set your Artifactory repository in the container. Dependencies will be fetched automatically from the Zextras Artifactory repositories.

For example, targeting Ubuntu Jammy:

```bash
echo "machine zextras.jfrog.io" >> auth.conf
echo "login $USERNAME" >> auth.conf
echo "password $SECRET" >> auth.conf
mv auth.conf /etc/apt
echo "deb [trusted=yes] https://zextras.jfrog.io/artifactory/ubuntu-devel jammy main" > zextras.list
mv zextras.list /etc/apt/sources.list.d/
```

#### For Community Contributors

First build dependencies from [carbonio-thirds](https://github.com/zextras/carbonio-thirds), then build this project with the `DEPS_DIR` option:

```bash
make build TARGET=ubuntu-jammy DEPS_DIR=../carbonio-thirds/artifacts
```

> **Note**: Use the same `TARGET` (ubuntu-jammy, ubuntu-noble, rocky-8, rocky-9) for both carbonio-thirds and carbonio-erlang.

### Supported Targets

- `ubuntu-jammy` - Ubuntu 22.04 LTS
- `ubuntu-noble` - Ubuntu 24.04 LTS
- `rocky-8` - Rocky Linux 8
- `rocky-9` - Rocky Linux 9

### Configuration

You can customize the build by setting environment variables:

```bash
# Use a specific container runtime
make build TARGET=ubuntu-jammy CONTAINER_RUNTIME=docker

# Use a different output directory
make build TARGET=rocky-9 OUTPUT_DIR=./my-packages
```

## Installation

These packages are distributed as part of the [Carbonio platform](https://zextras.com/carbonio). To install:

### Ubuntu (Jammy/Noble)

```bash
apt-get install <package-name>
```

### Rocky Linux (8/9)

```bash
yum install <package-name>
```

## Usage

After installation, the Erlang runtime and tools are available at `/opt/zextras/common/bin/`:

```bash
# Check Erlang version
/opt/zextras/common/bin/erl -version

# Check Elixir version
/usr/bin/elixir --version
```

The EPMD (Erlang Port Mapper Daemon) service is installed and can be managed via systemd:

```bash
# Start EPMD service
systemctl start epmd

# Enable EPMD service on boot
systemctl enable epmd
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for information on how to contribute to this project.

## License

This project is licensed under the GNU Affero General Public License v3.0 - see the [LICENSE.md](LICENSE.md) file for details.
