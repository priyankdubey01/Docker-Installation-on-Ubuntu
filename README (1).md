# Docker Installation on Ubuntu Using Bash Script

A simple Bash script to install Docker Engine on Ubuntu Linux using Docker's official APT repository. The script prepares the package manager, adds Docker's official GPG key and repository, installs Docker Engine and related plugins, and verifies the installation.

## Features

- Updates the APT package index.
- Installs required packages (`ca-certificates` and `curl`).
- Adds Docker's official GPG signing key.
- Configures Docker's official Ubuntu APT repository.
- Installs:
  - Docker Engine (`docker-ce`)
  - Docker CLI (`docker-ce-cli`)
  - `containerd.io`
  - Docker Buildx plugin
  - Docker Compose plugin
- Starts and verifies Docker, then runs the `hello-world` test container.

## Requirements

- An Ubuntu Linux system with a supported release.
- A user account with `sudo` privileges.
- An active internet connection.
- A review of existing container packages before removing or replacing them.

> **Warning:** The original script includes a command intended to remove existing Docker/container packages. Review it carefully before running, because removing packages may affect existing containers or workloads. Back up important data and configurations first.

## Files

```text
.
└── docker.sh
```

## Installation and Usage

### 1. Clone the repository

Replace `YOUR_USERNAME/YOUR_REPOSITORY` with your GitHub username and repository name.

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
cd YOUR_REPOSITORY
```

Alternatively, download `docker.sh` directly into a local directory.

### 2. Review the script

```bash
nano docker.sh
```

Before executing, ensure that any interactive instructions are comments. For example, change:

```bash
Press Ctrl + C
```

to:

```bash
# Press Ctrl + C if you intentionally need to stop a foreground command.
```

The script's `systemctl status docker` command is primarily for displaying service status; it is not necessary for installation. The script should explicitly start Docker and then run the test container.

### 3. Make the script executable

```bash
chmod +x docker.sh
```

### 4. Run the script

```bash
./docker.sh
```

The script uses `sudo` for system-level package and service operations. Read each command before approving it.

## Verify the Installation

Check the Docker version:

```bash
docker --version
```

Check the Docker service:

```bash
sudo systemctl status docker
```

If needed, start Docker:

```bash
sudo systemctl start docker
```

Run the official test image:

```bash
sudo docker run hello-world
```

A successful test prints a message confirming that Docker can pull and run a container.

Check Docker Compose:

```bash
docker compose version
```

Check Buildx:

```bash
docker buildx version
```

## Troubleshooting

### Docker service is not running

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### Permission denied when running Docker

Use `sudo` for the command, for example:

```bash
sudo docker ps
```

You can optionally add your user to the `docker` group, but membership in that group grants root-level privileges:

```bash
sudo usermod -aG docker "$USER"
```

Sign out and sign back in for the group change to take effect. Only do this if you understand the security implications.

### APT repository or package errors

Refresh the package index and review the official Docker installation instructions for your Ubuntu release:

https://docs.docker.com/engine/install/ubuntu/

## Security Notes

- Use Docker's official documentation as the source of truth for supported Ubuntu versions and installation steps.
- Do not run shell scripts with elevated privileges without reviewing their contents.
- Do not remove existing container packages on a production system without assessing dependencies and backups.
- Access to the Docker daemon can provide root-equivalent control of the host.

## Contributing

Issues and pull requests are welcome. Please include your Ubuntu version and relevant error output when reporting installation problems. Remove secrets and sensitive system information from logs before sharing them.

## License

Choose a license before publishing this project. If you intend to allow reuse, add a `LICENSE` file with the license you select.
