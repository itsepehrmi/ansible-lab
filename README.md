cat > README.md << 'ENDREADME'
# Ansible Lab

A local multi-server environment managed by Ansible — simulating AWS EC2 fleet management.

## What this project demonstrates

- Multi-server provisioning: 3 Ubuntu servers running as Docker containers
- Infrastructure as Code: Every server defined in Docker Compose
- Agentless automation: Ansible connects via SSH, no agent installed
- Passwordless authentication: SSH keys + sudo NOPASSWD
- Idempotent configuration: Running the playbook twice = same result
- Group management: All servers managed through one inventory

## Architecture

Ansible (control node) connects via SSH to 3 target servers:

    +-------------+ +-------------+ +-------------+
    | ec2-node-1 | | ec2-node-2 | | ec2-node-3 |
    | Ubuntu | | Ubuntu | | Ubuntu |
    | Nginx | | Nginx | | Nginx |
    | :22 -> 2201 | | :22 -> 2202 | | :22 -> 2203 |
    +-------------+ +-------------+ +-------------+
           ^ ^ ^
           | | |
           +-------------------+-------------------+
                               |
                        +--------------+
                        | Ansible |
                        +--------------+

## AWS Concept Mapping

| AWS Concept | This Project |
|----------------------|-------------------------------------------|
| EC2 Instances | 3 Docker containers |
| AMI | ansible-lab-node:latest |
| IAM Key Pair | ssh/id_lab |
| Security Groups | Docker port mappings (2201-2203) |
| Ansible Control Node | Your host machine |
| Target Servers | ec2-node-1, ec2-node-2, ec2-node-3 |

## Project Structure

    ansible-lab/
    |-- docker/
    | `-- Dockerfile
    |-- ssh/
    | |-- id_lab
    | `-- id_lab.pub
    |-- ansible/
    | |-- inventory.ini
    | `-- install-nginx.yml
    |-- docker-compose.yml
    `-- README.md

## Quick Start

Prerequisites:
- Docker + Docker Compose
- Ansible

Steps:

    git clone https://github.com/itsepehrmi/ansible-lab.git
    cd ansible-lab
    ssh-keygen -t ed25519 -f ./ssh/id_lab -N "" -C "ansible-lab"
    docker build -t ansible-lab-node:latest -f docker/Dockerfile .
    docker-compose up -d
    cd ansible
    ansible -i inventory.ini webservers -m ping
    ansible-playbook -i inventory.ini install-nginx.yml

## The Playbook

The playbook `install-nginx.yml` performs:

1. Updates apt cache (with 1-hour caching)
2. Installs Nginx
3. Deploys a custom index.html showing the server hostname
4. Ensures Nginx is running and enabled at boot

Handlers: Nginx is only restarted if the config or content changes.

## Idempotency Test

Run the playbook twice:

    ansible-playbook -i inventory.ini install-nginx.yml
    ansible-playbook -i inventory.ini install-nginx.yml

The second run reports changed=0 for all hosts, proving the playbook is idempotent.

## Key Concepts Demonstrated

- Inventory groups: [webservers] targets multiple hosts
- Become (privilege escalation): become: yes runs tasks as root
- Handlers: Restart services only when notified
- Templates/Variables: uses server hostname dynamically
- Task idempotency: Safe to run repeatedly

## License

MIT
ENDREADME
