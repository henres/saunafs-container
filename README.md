# leilfs-container
Experimental container-based deployment cluster for [LeilFS](https://github.com/leil-io/saunafs)

The ultimate goal of this repository is to create all advantages of containers into LeilFS project.

## Warning - about testing and educational usage only

This project was created for making fast DEMOs and playground purpose.

**It should NOT be used for production data!**

## Pre-built images (GitHub Packages)

Images are automatically built and published to the GitHub Container Registry on every push to `main`
and on every version tag (`v*.*.*`):

```
ghcr.io/leil-io/leilfs-container/leilfs-base
ghcr.io/leil-io/leilfs-container/leilfs-master
ghcr.io/leil-io/leilfs-container/leilfs-metalogger
ghcr.io/leil-io/leilfs-container/leilfs-chunkserver
ghcr.io/leil-io/leilfs-container/leilfs-cgiserver
ghcr.io/leil-io/leilfs-container/leilfs-client
```

Available tags: tag follow git repository tags

## Requirements

Project requires `docker` and `docker-compose`

Also some (`1GB`) free space on hdd is recommended for efficient simulation of storage replication.

## Usage

Clone the repository:

```shell
git clone https://github.com/leil-io/leilfs-container.git
cd leilfs-container
```

Builds use the public LeilFS APT repository and do not require credentials.

---

### Build and Run with Docker

> **Note:**  
> On some systems buildx docker plugin may need to be install prior following step. eg. ubuntu require
> ```shell
> sudo apt install -y docker-buildx
> ```

**Latest LeilFS version (default):**

```shell
# Build the shared base image
docker build \
  -f leilfs-base/Dockerfile \
  -t leilfs-base leilfs-base/

# Build and start all services
docker compose up --build
```

**Pinned LeilFS version** (e.g. `5.7.1`):

```shell
# Build the shared base image with a pinned version
docker build \
  --build-arg SAUNAFS_VERSION=5.7.1 \
  -f leilfs-base/Dockerfile \
  -t leilfs-base leilfs-base/

# Build all services with the same pinned version
docker compose build \
  --build-arg SAUNAFS_VERSION=5.7.1

docker compose up
```

### Build and Run with Podman

If you previously created a `./volumes` folder while using Docker, please delete it before deploying with Podman. This prevents permission issues that can occur due to differences in how Docker and Podman handle volume ownership.

```shell
# Build the shared base image (no credentials required)
podman build \
  -f leilfs-base/Dockerfile \
  -t leilfs-base leilfs-base/

# Build and start all services
podman-compose up --build
```

Pass `--build-arg SAUNAFS_VERSION=5.7.1` to both commands to pin a specific version.

> **Note:**  
> `docker compose` (v2) or `podman-compose` is recommended.

---

Visit [http://localhost:29425/sfs.cgi?masterhost=master&masterport=9421](http://localhost:29425/sfs.cgi?masterhost=master&masterport=9421) to access the LeilFS CGI.

## Data Persistence and Initialization

This Docker deployment is designed for ease of use and demonstration.
- **No Pre-committed Data**: The `volumes/` directory is no longer part of this repository.
- **Automatic Initialization**: On first startup, each service (master, metalogger, chunkservers) will automatically:
    - Create necessary configuration files using defaults from the LeilFS packages (found in `/usr/share/doc/leilfs-*/examples/` within the containers).
    - Initialize their respective data directories.
- **Persistent Data**: If you map Docker volumes to the standard LeilFS data and configuration paths (e.g., `/var/lib/saunafs/`, `/etc/saunafs/`), your data and custom configurations will persist across container restarts. If these mapped volumes are empty on first start, they will be initialized as described above.
- **Chunkserver Storage**:
    - Chunkservers will look for mount points at `/mnt/hdd001`, `/mnt/hdd002`, etc.
    - If you provide external volumes mounted to these paths in your `docker-compose.yml`, they will be used.
    - If these paths are not externally mounted, the startup script will create them as directories within the container (volatile storage) and issue a warning. This is suitable for testing but not for production data.

This setup ensures that you can get a LeilFS cluster running quickly without manual configuration steps, while still allowing for persistent storage and custom configurations when needed.

## Cleaning Up Data

If you have used Docker named volumes or host-mounted directories (e.g., by customizing `docker-compose.yml` to map local paths like `./volumes/master/data:/var/lib/saunafs`), your LeilFS data will persist even after containers are stopped and removed.

To completely reset the LeilFS environment and start fresh, you will need to remove this persistent data. 

- **If using Docker named volumes**: You can list them with `docker volume ls` and remove them with `docker volume rm <volume_name>`.
- **If using host-mounted directories**: For example, if you created a local `volumes` directory in your project and mapped subdirectories from it (e.g., `volumes/master/data`, `volumes/chunkserver1/hdd001`, etc.), you would need to manually delete these local directories. 

  **Example for host-mounted `./volumes/` directory:**
  If you had a structure like:
  ```
  your-project-root/
    docker-compose.yml
    volumes/
      master/
        etc/
        var_lib/
      chunkserver1/
        hdd001/
        hdd002/
      ...
  ```
  You would remove the data by deleting the `volumes` directory from your host machine:
  ```shell
  # WARNING: This command permanently deletes data!
  # Ensure you are in your project root and understand the consequences.
  sudo rm -r ./volumes
  ```
  **Be extremely careful with `rm -r` commands.** Double-check the path to ensure you are deleting the correct directory. Incorrect usage can lead to irreversible data loss on your system.

After cleaning up persistent data, the next `docker compose up` will re-initialize everything from scratch using the default configurations.
