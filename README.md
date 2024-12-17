# Story Node Ansible

Automated Ansible deployment for Story consensus and Geth nodes. This repository provides a complete setup for deploying and managing Story nodes across multiple servers.

## Features

- Automated deployment of Story consensus and Geth nodes
- Cosmovisor setup for automatic upgrades
- JWT authentication between Story and Geth
- Systemd service management
- Environment configuration
- Node monitoring setup

## Directory Structure

```
story-node-ansible/
├── inventory.yml                # Inventory configuration
├── group_vars/
│   └── all.yml                 # Global variables
├── roles/
│   ├── common/                 # Common setup tasks
│   │   └── tasks/
│   │       └── main.yml
│   ├── jwt-setup/              # JWT authentication setup
│   │   └── tasks/
│   │       └── main.yml
│   ├── cosmovisor/             # Cosmovisor installation and setup
│   │   └── tasks/
│   │       └── main.yml
│   ├── story-consensus/        # Story consensus node setup
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── templates/
│   │       └── config.toml.j2
│   └── story-geth/            # Geth node setup
│       ├── tasks/
│       │   └── main.yml
│       └── templates/
│           └── geth.service.j2
├── templates/                  # Global service templates
│   ├── story.service.j2
│   └── story-geth.service.j2
└── site.yml                   # Main playbook
```

## Prerequisites

1. **Ansible Installation**
```bash
sudo apt update
sudo apt install ansible -y
```

2. **SSH Access**
- Configure SSH key authentication for target nodes
```bash
ssh-copy-id -i ~/.ssh/your_key.pem ubuntu@target-host
```

3. **Target Node Requirements**
- Ubuntu 20.04 or later
- Minimum 6 cores
- 16GB RAM
- 400GB NVME storage
- 100 Mb/s network

## Configuration

### 1. Inventory Setup

Create `inventory.yml` with your target hosts:

```yaml
all:
  vars:
    ansible_user: ubuntu
    ansible_ssh_private_key_file: ~/.ssh/your_key.pem
  children:
    story_nodes:
      children:
        consensus_nodes:
          hosts:
            consensus1:
              ansible_host: your_consensus_ip
        geth_nodes:
          hosts:
            geth1:
              ansible_host: your_geth_ip
```

### 2. Variables Configuration

Update `group_vars/all.yml`:

```yaml
go_version: "1.22.3"
story_version: "v0.13.0"
geth_version: "v0.11.0"
chain_id: "odyssey-0"
moniker: "your-node-name"
story_port: "52"
home_dir: "/home/{{ ansible_user }}/.story"
```

## Installation

1. **Clone the Repository**
```bash
git clone <repository-url>
cd story-node-ansible
```

2. **Run the Playbook**
```bash
ansible-playbook -i inventory.yml site.yml
```

3. **Verify Installation**
```bash
# Check Story service
ansible story_nodes -i inventory.yml -m shell -a "systemctl status story" --become

# Check Geth service
ansible story_nodes -i inventory.yml -m shell -a "systemctl status story-geth" --become
```

## Service Management

### Start Services
```bash
ansible story_nodes -i inventory.yml -m systemd -a "name=story-geth state=started" --become
ansible story_nodes -i inventory.yml -m systemd -a "name=story state=started" --become
```

### Stop Services
```bash
ansible story_nodes -i inventory.yml -m systemd -a "name=story state=stopped" --become
ansible story_nodes -i inventory.yml -m systemd -a "name=story-geth state=stopped" --become
```

### View Logs
```bash
# Story logs
ansible story_nodes -i inventory.yml -m shell -a "journalctl -fu story -o cat" --become

# Geth logs
ansible story_nodes -i inventory.yml -m shell -a "journalctl -fu story-geth -o cat" --become
```

## Troubleshooting

### Common Issues

1. **Service Start Failures**
- Check logs using journalctl
- Verify JWT file existence and permissions
- Ensure correct binary paths

2. **Cosmovisor Issues**
- Verify environment variables
- Check binary permissions
- Validate directory structure

3. **Network Issues**
- Verify firewall rules
- Check external IP configuration
- Validate peer connections

### Validation Commands

```bash
# Check binary installations
ansible story_nodes -i inventory.yml -m shell -a "which story geth cosmovisor" --become

# Verify configurations
ansible story_nodes -i inventory.yml -m shell -a "cat {{ home_dir }}/story/config/config.toml" --become

# Check JWT setup
ansible story_nodes -i inventory.yml -m shell -a "ls -l {{ home_dir }}/geth/odyssey/jwtsecret" --become
```

## Maintenance

### Upgrading Nodes

1. Update versions in `group_vars/all.yml`
2. Run the playbook with upgrade tag:
```bash
ansible-playbook -i inventory.yml site.yml --tags upgrade
```

### Backup

1. Configuration backup:
```bash
ansible story_nodes -i inventory.yml -m fetch -a "src={{ home_dir }}/story/config/config.toml dest=backups/"
```

2. JWT backup:
```bash
ansible story_nodes -i inventory.yml -m fetch -a "src={{ home_dir }}/geth/odyssey/jwtsecret dest=backups/"
```

## Security Considerations

- Always use SSH key authentication
- Keep JWT files secure
- Regularly update node software
- Monitor system resources
- Use proper firewall rules

## License

[Insert License Information]

## Contributing

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## Support

- Open issues on GitHub
- Join the Story Discord community
- Check documentation at [Story docs]

---

**Note:** Always test in a staging environment before deploying to production.