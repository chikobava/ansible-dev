# Ansible Devcontainer Workbench

This repository serves as a practical guide to using **devcontainers** for Ansible development. It demonstrates how to quickly spin up a consistent, cross-platform development environment on **macOS**, **Windows**, or **Linux** using Visual Studio Code and Docker.

## Prerequisites

You’ll need:

- [Visual Studio Code](https://code.visualstudio.com/)
- [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- [Docker](https://www.docker.com/products/docker-desktop/) installed and running on your system

## What's Inside

We use a custom `Dockerfile` based on the official [Ansible community dev tools image](https://ansible.readthedocs.io/projects/dev-tools/), which comes with all the essential tools for developing and testing playbooks and roles.

### Dockerfile

```Dockerfile
FROM ghcr.io/ansible/community-ansible-dev-tools:latest

ARG USERNAME=ansible
ARG UUID=1000

RUN groupadd -g $UUID $USERNAME && \
    useradd -u $UUID -g $UUID -m $USERNAME -s /bin/bash

USER ansible
```

- Uses a prebuilt image with Ansible tools
- Adds a non-root ansible user for development safety

### .devcontainer/devcontainer.json

```json
{
  "name": "ansible-dev",                         // 1
  "dockerFile": "Dockerfile",                    // 2
  "context": "..",                               // 3
  "remoteUser": "ansible",                       // 4
  "workspaceFolder": "/workspace",               // 5
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind,consistency=delegated", // 6
  "runArgs": ["-h", "ansible-dev"],              // 7
  "customizations": {
    "vscode": {
      "extensions": [
        "redhat.ansible"                         // 8
      ]
    }
  }
}
```

### Breakdown

1. **`name`** – Prefix for the container name (e.g., `vsc-ansible-dev-<hash>`)
2. **`dockerFile`** – Uses the local `Dockerfile` for image build
3. **`context`** – Sets the build context to the parent workspace directory
4. **`remoteUser`** – Connects to the container as the non-root `ansible` user
5. **`workspaceFolder`** – Container workspace directory (`/workspace`)
6. **`workspaceMount`** – Mounts your local VSCode folder into the container
7. **`runArgs`** – Sets hostname inside the container to `ansible-dev`
8. **`customizations.vscode.extensions`** – Installs the RedHat Ansible extension for linting, syntax highlighting, etc.

## Getting Started
To launch the devcontainer:

Clone this repository:

```bash
git clone git@github.com:chikobava/ansible-dev.git
cd ansible-dev
```

Open the project in VSCode:
```
code .
```

When prompted (bottom right), click **"Reopen in Container"**.  
Alternatively, you can press `Ctrl + Shift + P` to open the VSCode command palette and type **"Rebuild Container"** to rebuild and reopen the devcontainer anytime.

If you have any troubles, check out the official VSCode documentation on running devcontainers [here](https://code.visualstudio.com/docs/devcontainers/containers).


VSCode will build and launch the devcontainer. Within moments, you'll be inside a fully-featured Ansible development environment. Start creating your playbooks or adjust the inventory to match your infrastructure.

### Project Structure and Examples

This repository uses the **alternative directory layout** recommended by [Ansible best practices](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html#alternative-directory-layout).

#### 📁 Inventory

Inventory files are organized under the `inventories/` directory. For example, the development environment inventory is located at:
```
inventories/
└── dev/
    └── inventory.yml
```

#### 📁 Playbooks

Example playbooks are stored in the `playbooks/` directory. A simple ping test playbook is available at:
```
playbooks/
└── ping.yml
```

These examples serve as a starting point for testing your Ansible setup inside the development container. You can modify them or add new environments/playbooks as needed.

### SSH Key Access in the devcontainer

To allow Ansible to connect to remote systems using your SSH key, you have two main options:

---

#### ✅ Recommended: Use `ssh-agent` (clean and secure)

1. If you're on **Windows**, follow the [Microsoft guide](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement) to install and configure `ssh-agent` to run as a service. 

    **Note:** On Linux (+WSL) and macOS, before adding your SSH key with `ssh-add`, you may need to start the SSH agent by running:
    ```bash
    eval "$(ssh-agent -s)"
    ```
    This command initializes the ssh-agent process in your shell session, enabling key forwarding to the devcontainer. Ensure that you run the `code` command from the same shell session where you started your ssh-agent process. 

2. Add your private key to the agent before starting the container (on your host machine, e.g. laptop):
    ```bash
    ssh-add ~/.ssh/id_rsa
    ```
3. To verify loaded keys:
    ```bash
    ssh-add -l
    ```
Once your ssh-agent is running and your key is added, VS Code will forward your SSH keys into the devcontainer automatically. To verify, simply run `ssh-add -l` in the container.
No need to mount or copy anything — Ansible inside the container will have access to your forwarded SSH keys.

#### 🛠️ Alternative: Manually mount your SSH key (if ssh-agent isn’t an option)
You can bind-mount your SSH private key into the container by adding the following line to your `.devcontainer/devcontainer.json` file under the "mounts" section:

```json
"mounts": [
  "type=bind,source=${env:HOME}/.ssh/id_rsa,target=/ansible/.ssh/id_rsa"
]
```
⚠️ Important notes:

- Ensure correct file paths depending on your OS.
- Be cautious with file permissions — private keys must be chmod 600.
- Only use this method if you're aware of the security implications.

In general, using `ssh-agent` is the safest and most seamless method.
