# Configuration Management with Ansible

This section of the project demonstrates the use of **Ansible** to automate server configuration on an AWS EC2 instance provisioned in Part 1.

## Tools Used

- **Ansible**: A powerful automation engine used for IT tasks such as configuration management and application deployment.
- **Docker**: Containerization tool installed via Ansible on the target EC2 instance.
- **Ubuntu 22.04**: The operating system of the EC2 instance.

## Ansible Playbook Overview

Here’s a breakdown of the Ansible playbook used to configure the EC2 server.

### Playbook File: `docker-setup.yml`

```yaml
- name: Install Docker and Deploy App
  hosts: web
  become: true

  tasks:
    - name: Update APT
      apt:
        update_cache: yes
```

- This task refreshes the package index on the Ubuntu EC2 instance to make sure we have access to the latest available packages.

```yaml
- name: Install Docker
  apt:
    name: docker.io
    state: present
```

- Installs the docker.io package if it’s not already present. This ensures Docker is installed on the target instance.

```yaml
- name: Enable and start Docker
  systemd:
    name: docker
    enabled: yes
    state: started
```

- Ensures the Docker service is started and enabled on boot using systemd. This guarantees Docker will persist even after reboots.

```yaml
- name: Remove old app files (clean cache)
  file:
    path: /home/ubuntu/app
    state: absent
```

- Deletes any old app files on the EC2 instance to avoid using outdated content.

```yaml
- name: Copy app files to EC2
  copy:
    src: ./app/
    dest: /home/ubuntu/app
    mode: 0755
```

- Copies the latest version of your application (e.g., HTML, Dockerfile, etc.) from your local directory to the EC2 instance under /home/ubuntu/app.

```yaml
- name: Build Docker image (no cache)
  shell: docker build --no-cache -t my-web-app /home/ubuntu/app
  args:
    chdir: /home/ubuntu
```

- Builds a new Docker image named my-web-app from the copied files. --no-cache ensures that the image is always built from scratch, reflecting all file changes.

```yaml
- name: Remove existing container if exists
  shell: docker rm -f static-web || true
```

- Force-removes the existing Docker container named static-web (if any), allowing the next step to run cleanly.

```yaml
- name: Run Docker container
  shell: docker run -d -p 80:80 --name static-web my-web-app || true
```

- Runs a new container from the built image and maps port 80 of the container to port 80 of the host. This allows the web app to be accessible via the EC2 public IP.

## Troubleshooting Tips

1. **SSH Failures**

   - Make sure `key.pem` has correct permissions: `chmod 600 key.pem`
   - Ensure your IP is allowed in EC2 Security Groups (port 22 and 80).

2. **Docker Issues**

   - Ensure Docker daemon is running: `sudo systemctl start docker`
   - Check logs: `docker logs static-web`

3. **Port Conflicts**
   - Error: `Bind for 0.0.0.0:80 failed`
     - Another process may be using port 80. Stop it or use a different port.

## Summary

This Ansible playbook ensures that the EC2 instance is consistently configured with Docker and the latest version of the application. By automating this process, we eliminate manual setup steps and ensure uniform deployment across environments.
