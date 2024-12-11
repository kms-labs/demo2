# Learn how to populate a VOLUME

---

## Introduction

In this guide, you will learn to:

1. Create a Docker image with Nginx serving static content for Volume demos.
2. Populate a Docker volume with read-write and read-only permissions using:
   - `--mount` flag
   - `-v` flag
3. Manage Docker volumes for data persistence and security.

Docker volumes persist data across container restarts and enable sharing between containers.

---

## Step 1: Create a Docker Image with Nginx Static Content

### Step 1.1: Review Dockerfile

**Dockerfile**

```dockerfile
# Use nginx:alpine-slim as base Docker Image
FROM nginx:alpine-slim

# OCI Labels
LABEL org.opencontainers.image.authors="KMS Healthcare"
LABEL org.opencontainers.image.title="Demo: Populate Docker Volumes with Containers"
LABEL org.opencontainers.image.description="A Dockerfile demo illustrating how to populate Docker volumes using containers and serving static content with NGINX."
LABEL org.opencontainers.image.version="1.0"

# Using COPY to copy local static content to Nginx HTML directory
COPY ./static-content/ /usr/share/nginx/html
```

**Explanation:**

- **FROM nginx:alpine-slim:** Uses the lightweight Alpine-based Nginx image.
- **LABELs:** Provide metadata about the image.
- **COPY ./static-content/ /usr/share/nginx/html:** Copies all files from the `static-content` directory on the host to Nginx's default HTML directory inside the container.

### Step 1.2: Review Static Content

**Directory Structure:**

```
Dockerfiles/
├── Dockerfile
└── static-content/
    ├── app1/
    │   └── index.html
    ├── file1.html
    ├── file2.html
    ├── file3.html
    ├── file4.html
    ├── file5.html
    └── index.html
```


### Step 1.3: Build a Docker Image
```bash
# Change directory
cd $(git rev-parse --show-toplevel)/a17-Docker-VOLUMES-using-containers/Dockerfiles

# Build the Docker image
docker build -t mynginx-static:v1 .

# Run Docker Container and Verify
docker run --name=volumes-demo-base-container -p 8090:80 -d mynginx-static:v1

# List Docker Containers
docker ps -f "name=demo"

# Verify docker image files/directories
docker run --rm -it mynginx-static:v1 sh -c "ls -laR /usr/share/nginx/html"

# Observation:
# We have all our static content present and accessible
```

Access Application http://localhost:8090

---

## Step 2: Populate a Volume Using Container

Populating a Docker volume involves mounting it to a container's directory. If the directory contains data, Docker copies it into the volume, ensuring the volume starts with the container's initial data.

### OPTION 1: Using `--mount` with RW access

```bash
# Single Line Format
docker run --name volume-demo1 -p 8091:80 --mount type=volume,source=myvol103,target=/usr/share/nginx/html -d mynginx-static:v1

# Verify volume
docker volume ls
docker run --rm -i -v=myvol103:/tmp/myvolume busybox ls -laR /tmp/myvolume

docker volume inspect myvol103 | jq
```

**Explanation:**

- `--name volume-demo1`: Names the container `volume-demo1`.
- `-p 8091:80`: Maps host port `8091` to container port `80`.
- `--mount type=volume,source=myvol103,target=/usr/share/nginx/html`: Mounts the named volume `myvol103` to `/usr/share/nginx/html` inside the container.
- `-d mynginx-static:v1`: Runs the container in detached mode using the `mynginx-static:v1` image.

#### Inspect the Docker Container

```bash
# Pretty-Print Mounts Information Using jq
docker inspect --format='{{json .Mounts}}' volume-demo1 | jq
```

**Explanation:**

- These commands provide detailed information about the container's mounts, showing that `myvol103` is correctly mounted to `/usr/share/nginx/html`.

Access via Browser http://localhost:8091

```bash
# Observation:
# - All static content is accessible, confirming that the volume has been populated correctly.
```

### OPTION 2: Using `-v` with RW access

The `-v` or `--volume` flag offers a shorthand syntax for mounting volumes.

```bash
# Single Line Format
docker run --name volume-demo3 -p 8093:80 -v myvol103:/usr/share/nginx/html -d nginx:alpine-slim
```

**Explanation:**

- `-v myvol103:/usr/share/nginx/html`: Mounts the named volume `myvol103` to `/usr/share/nginx/html` inside the container with read-write permissions.

#### Verify the Volume Mount

```bash
# List Docker Containers
docker ps -f "name=demo"

# Check Disk Usage
docker exec -it volume-demo3 /bin/sh -c "df -h | grep html"

# List the Mounted Directory
docker exec -it volume-demo3 /bin/sh -c "ls -laR /usr/share/nginx/html"

# Observations:
# 1. The volume `myvol103` is mounted to `/usr/share/nginx/html`.
# 2. The static content is present and accessible within the container.
```

#### Modify Content (RW access)

```bash
# Connect to the Container
docker exec -it volume-demo3 /bin/sh

# Inside the Container: Copy a File
cd /usr/share/nginx/html
cp index.html kms.html

# Exit the Container Shell
exit

# Observation:
# - The file `kms.html` is successfully created and accessible, indicating that the volume is mounted with read-write permissions.
```

Access the New File via Browser http://localhost:8093/kms.html

### OPTION 3: Using `--mount` with RO access

```bash
# Single Line Format
docker run --name volume-demo4 -p 8094:80 --mount source=myvol103,target=/usr/share/nginx/html,readonly -d nginx:alpine-slim
```

**Explanation:**

- `readonly`: Mounts the volume as read-only, preventing any write operations within the container.

#### Verify Read-Only Mount

```bash
# Connect to the Container
docker exec -it volume-demo4 /bin/sh

# Inside the Container: Attempt to Copy a File
cd /usr/share/nginx/html
cp index.html healthcare.html

# Expected Output:
# cp: can't create 'healthcare.html': Read-only file system

# Exit the Container Shell
exit

# Observations:
# 1. Attempting to create or modify files within `/usr/share/nginx/html` fails due to read-only permissions.
# 2. The static content remains accessible but immutable.
```

Access the Application in Browser http://localhost:8094

### OPTION 4: Using `-v` with RO access

```bash
# Single Line Format
docker run --name volume-demo5 -p 8095:80 -v myvol103:/usr/share/nginx/html:ro -d nginx:alpine-slim
```

**Explanation:**

- `-v myvol103:/usr/share/nginx/html:ro`: Mounts the named volume `myvol103` to `/usr/share/nginx/html` inside the container with read-only permissions.

#### Verify Read-Only Mount

```bash
# Connect to the Container
docker exec -it volume-demo5 /bin/sh

# Inside the Container: Attempt to Copy a File
cd /usr/share/nginx/html
cp index.html healthcare.html

# Expected Output:
# cp: can't create 'healthcare.html': Read-only file system

# Exit the Container Shell
exit

# Observation:
# - Similar to `volume-demo4`, attempting to modify files results in an error, ensuring the volume is mounted as read-only.
```

Access the Application in Browser http://localhost:8095/kms.html

---

## Step 3: Clean-Up - KEEP THE VOLUME (myvol103) will be re-used in next demo#21

After completing the demos, it's important to clean up Docker resources to free up system resources and maintain a tidy environment.

```bash
# Delete All Containers
docker rm -f $(docker ps -aq -f "name=demo")

# Delete All Docker Images
docker rmi $(docker image ls -qa --filter 'reference=*nginx*')
```

---

## Conclusion

You have successfully:

- Created a Docker image with Nginx serving static content.
- Populated Docker volumes with read-write and read-only permissions using `--mount` and `-v` flags.
- Verified the integrity and permissions of mounted volumes.

Docker volumes manage persistent data, ensuring it persists across container lifecycles and enables data sharing among containers.

---

## Additional Notes

- **`--mount` vs. `-v`:**

  - **`--mount`:**
    - More verbose and recommended for complex setups.
    - Syntax: `--mount type=volume,source=<VOLUME_NAME>,target=<CONTAINER_PATH>,readonly`.

  - **`-v`:**
    - Shorthand for simple mounts.
    - Syntax: `-v <VOLUME_NAME>:<CONTAINER_PATH>:ro`.

- **Volume Persistence:**
  - Data persists after container removal, stored outside the container’s writable layer for integrity and isolation.

- **Best Practices:**
  - Use named volumes for clarity.
  - Mount data as read-only when it should not be modified.
  - Clean up unused volumes regularly.

- **Security Considerations:**
  - Mount volumes with proper permissions for better security.
  - Avoid mounting sensitive files unless necessary.

---
