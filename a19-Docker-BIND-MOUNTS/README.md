# Learn with Bind Mounts

---

## Introduction

In this guide, you will learn to:

1. **Mount a host directory** to a Docker container using bind mounts.
2. **Use bind mounts with read-write permissions** with `--mount` and `-v` flags.
3. **Verify the integrity and accessibility** of mounted directories.
4. **Clean up Docker resources** after completion.

Bind mounts are useful for allowing containers to access or modify host files, such as in development or for sharing configurations.

---

## Step 1: Prepare Host Directory with Static Content

Before mounting a directory to a Docker container, ensure that the host directory exists and contains the necessary static content.

```bash
# List directory
ls -laR $(git rev-parse --show-toplevel)/a19-Docker-BIND-MOUNTS/myfiles
```
---

## Step 2: Using `--mount` with RW access

The `--mount` flag provides a clear and explicit syntax for bind mounts.

### Run Docker Container with Bind Mount Using `--mount`

```bash
# Change directory
cd $(git rev-parse --show-toplevel)/a19-Docker-BIND-MOUNTS/myfiles

# Single-Line Format
docker run --name bind-demo1 -p 8091:80 --mount type=bind,source="$(pwd)"/static-content,target=/usr/share/nginx/html -d nginx:alpine-slim
```

**Explanation:**

- **`--name bind-demo1`**: Names the container `bind-demo1`.
- **`-p 8091:80`**: Maps host port `8091` to container port `80`.
- **`--mount type=bind,source="$(pwd)"/static-content,target=/usr/share/nginx/html`**:
  - **`type=bind`**: Specifies a bind mount.
  - **`source="$(pwd)"/static-content`**: The directory on the host machine to mount.
  - **`target=/usr/share/nginx/html`**: The directory inside the container where the host directory will be mounted.
- **`-d nginx:alpine-slim`**: Runs the container in detached mode using the Nginx Alpine image.

### Verify the Bind Mount

```bash
# List Docker Containers
docker ps -f "name=demo"

# Check Disk Usage
docker exec -it bind-demo1 /bin/sh -c "df -h | grep html"

# Attempt to Create a New File (Read-Write)
docker exec -it bind-demo1 /bin/sh -c "cd /usr/share/nginx/html/ && cp index.html kms.html"

# List the Mounted Directory
docker exec -it bind-demo1 /bin/sh -c "ls -laR /usr/share/nginx/html"
```

**Observations:**

1. **Mount Point Verification:**
   - `/usr/share/nginx/html` is mounted to the `static-content` directory on the host.
   - Running `df -h | grep html` should show the mount point details.

2. **Data Integrity:**
   - The `static-content` directory's contents are accessible within the container.
   - Creating a new file (`kms.html`) demonstrates read-write access.

3. **Accessing the Application:**
   - Open a browser and navigate to http://localhost:8091 to view the static content.
   - Access http://localhost:8091/kms.html to verify the newly created file.

### Inspect the Docker Container

```bash
# Pretty-Print Mounts Information Using jq
docker inspect --format='{{json .Mounts}}' bind-demo1 | jq
```

**Explanation:**

- **`Type`**: Indicates a bind mount.
- **`Source`**: The directory on the host machine.
- **`Destination`**: The directory inside the container where the bind mount is applied.
- **`RW`**: `true` denotes read-write access.

---

## Step 3: Using `-v` with RW access

The `-v` or `--volume` flag provides a shorthand syntax for bind mounts.

### Run Docker Container with Bind Mount Using `-v`

```bash
# Change directory
cd $(git rev-parse --show-toplevel)/a19-Docker-BIND-MOUNTS/myfiles

# Single-Line Format
docker run --name bind-demo2 -p 8092:80 -v "$(pwd)"/static-content:/usr/share/nginx/html -d nginx:alpine-slim
```

**Explanation:**

- **`--name bind-demo2`**: Names the container `bind-demo2`.
- **`-p 8092:80`**: Maps host port `8092` to container port `80`.
- **`-v "$(pwd)"/static-content:/usr/share/nginx/html`**:
  - **`$(pwd)/static-content`**: The directory on the host machine to mount.
  - **`/usr/share/nginx/html`**: The directory inside the container where the host directory will be mounted.
- **`-d nginx:alpine-slim`**: Runs the container in detached mode using the Nginx Alpine image.

### Verify the Bind Mount

```bash
# List Docker Containers
docker ps -f "name=demo"

# Check Disk Usage
docker exec -it bind-demo2 /bin/sh -c "df -h | grep html"

# Attempt to Create a New File (Read-Write)
docker exec -it bind-demo2 /bin/sh -c "cd /usr/share/nginx/html/ && cp index.html kms2.html"

# List the Mounted Directory
docker exec -it bind-demo2 /bin/sh -c "ls -laR /usr/share/nginx/html"
```

**Observations:**

1. **Mount Point Verification:**
   - `/usr/share/nginx/html` is mounted to the `static-content` directory on the host.
   - Running `df -h | grep html` should show the mount point details.

2. **Data Integrity:**
   - The `static-content` directory's contents are accessible within the container.
   - Creating a new file (`kms2.html`) demonstrates read-write access.

3. **Accessing the Application:**
   - Open a browser and navigate to http://localhost:8092 to view the static content.
   - Access http://localhost:8092/kms2.html to verify the newly created file.

### Step 3.3: Inspect the Docker Container

```bash
# Pretty-Print Mounts Information Using jq
docker inspect --format='{{json .Mounts}}' bind-demo2 | jq
```

**Explanation:**

- **`Type`**: Indicates a bind mount.
- **`Source`**: The directory on the host machine.
- **`Destination`**: The directory inside the container where the bind mount is applied.
- **`RW`**: `true` denotes read-write access.

---

## Step 4: Verify Files in Local Directory

After performing operations inside the containers, it's crucial to verify that changes are reflected on the host machine.

```bash
# List Files
ls -laR $(git rev-parse --show-toplevel)/a19-Docker-BIND-MOUNTS/myfiles/static-content
```

**Observation:**

- You should find the files `kms.html` and `kms2.html` present.
  - **`kms.html`**: Created by `bind-demo1`.
  - **`kms2.html`**: Created by `bind-demo2`.

**Explanation:**

- Changes in the container's mounted directory are instantly reflected on the host, showcasing the effectiveness of bind mounts with read-write permissions.

---

## Step 5: Clean-Up

After completing the demonstrations, it's important to clean up Docker resources to free up system resources and maintain a tidy environment.

```bash
# Delete All Containers
docker rm -f $(docker ps -aq -f "name=demo")

# Delete All Docker Images
docker rmi $(docker image ls -qa --filter 'reference=*nginx*')

# List Docker Volumes
docker volume ls

# Observation:
# Volumes will persist and are not deleted even after deleting containers or images
```

---

## Conclusion

You have successfully:

- **Mounted host directories** to Docker containers using `--mount` and `-v` flags.
- **Used bind mounts with read-write permissions** for container access and modification of host files.
- **Verified integrity and accessibility** by creating files in containers and observing changes on the host.
- **Cleaned up Docker resources** by removing containers, images.

Bind mounts enable seamless integration between host systems and containers, ideal for development, configuration management, and data persistence.

---

## Additional Notes

- **`--mount` vs. `-v`:**

  - **`--mount`:**
    - More explicit, recommended for complex setups.
    - **Syntax:** `--mount type=bind,source=<HOST_DIR>,target=<CONTAINER_DIR>`.

  - **`-v`:**
    - Shorthand for simple mounts.
    - **Syntax:** `-v <HOST_DIR>:<CONTAINER_DIR>`.

- **Security Considerations:**
  - Mount sensitive directories cautiously and set proper permissions to prevent unauthorized access.

- **Best Practices:**
  - Use absolute paths for host directories.
  - Regularly manage mounted directories for security.
  - Prefer named volumes for persistent data.

---
