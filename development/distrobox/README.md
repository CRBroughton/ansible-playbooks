# Distrobox Ansible Playbook

This Ansible playbook automates the creation of a Distrobox container and executes additional playbooks inside the containeridsed environment.

## Prerequisites

- **Distrobox** installed on your system
- **Ansible** installed on the host system
- **Podman** or **Docker** as the container runtime

### Installing Distrobox

```bash
# On Fedora/RHEL
sudo dnf install distrobox

# On Ubuntu/Debian
sudo apt install distrobox
```

## Configuration

The playbook includes preconfigured environments that you can select when running it.

### Available Environments

- **default**: Basic container based on Ubuntu 22.04
- **laravel**: PHP, Composer, Laravel, and Node.js via Volta
- **go**: Go programming language
- **nuxt**: A Nuxt focused container with Bun & Volta
- **elixir**: Elixir, Erlang, Hex, and the Phoenix framework

### Selecting an Environment

```bash
# Use default environment
ansible-playbook distrobox-installer.yml

# Use Laravel environment
ansible-playbook distrobox-installer.yml -e env_name=laravel
```
## Usage

### Basic Usage

```bash
# Run with default environment
ansible-playbook distrobox-installer.yml

# Run with Laravel environment
ansible-playbook distrobox-installer.yml -e env_name=laravel
```

The playbook will show you all available environments at the start and which one you've selected.

### Adding New Environments

To add a new environment, edit the `environments` section in the playbook:

```yaml
environments:
  # ... existing environments ...
  
  python_ds:
    name: "Python Data Science"
    description: "Python, Jupyter, pandas, numpy, and common DS libraries"
    container_name: "python-ds"
    container_image: "python:3.11"
    wait_time: 60
    additional_playbooks:
      - "install-python-ds.yml"
      - "setup-jupyter.yml"
```

Then use it with:
```bash
ansible-playbook distrobox-installer.yml -e env_name=python_ds
```

## Preconfigured Environment Playbooks

Each environment uses specific setup and completion playbooks:

### Default Environment
- **Setup**: `setup.yml`, `configure.yml`

### Laravel Environment  
- **Setup**: `install-laravel.yml`, `install-volta.yml`
- **Completion**: `completion-laravel.yml`

### Go Environment
- **Setup**: `install-go.yml`  
- **Completion**: `completion-go.yml`

### Nuxt Environment
- **Setup**: `install-volta.yml`, `install-bun.yml`
- **Completion**: `completion-nuxt.yml`

### Elixir Environment
- **Setup**: `install-elixir.yml`  
- **Completion**: `completion-elixir.yml`

The completion playbooks display environment-specific instructions and next steps after the container is set up.

## What the Playbook Does

1. **Verification**: Checks if Distrobox is installed
2. **Container Creation**: Creates a new Distrobox container with specified image
3. **Initialization**: Starts the container and waits for it to be ready
4. **Ansible Installation**: Automatically installs Ansible inside the container
5. **Playbook Execution**: Runs your additional playbooks inside the container
6. **Reporting**: Provides status updates and completion summary

## Additional Playbooks

The playbooks listed in `additional_playbooks` should be located in the installers directory or provide full paths. They will be copied to the container's home directory and executed.

Example additional playbooks:

### `setup.yml` - Basic setup
```yaml
---
- name: Basic container setup
  hosts: localhost
  tasks:
    - name: Update package cache
      package:
        update_cache: yes
      become: yes
    
    - name: Install essential packages
      package:
        name:
          - git
          - vim
          - curl
          - wget
        state: present
      become: yes
```

### `configure.yml` - User configuration
```yaml
---
- name: Configure user environment
  hosts: localhost
  tasks:
    - name: Create development directory
      file:
        path: "{{ ansible_env.HOME }}/dev"
        state: directory
    
    - name: Set git configuration
      git_config:
        name: "{{ item.name }}"
        value: "{{ item.value }}"
        scope: global
      loop:
        - { name: "user.name", value: "Your Name" }
        - { name: "user.email", value: "your@email.com" }
```

## Accessing Your Container

After the playbook completes, you can access your container:

```bash
# Enter the container
distrobox enter [container_name]

# List all containers
distrobox list

# Stop a container
distrobox stop [container_name]

# Remove a container
distrobox rm [container_name]
```