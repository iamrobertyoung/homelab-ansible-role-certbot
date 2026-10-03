# homelab-ansible-role-certbot

Ansible role for automated SSL certificate management using Let's Encrypt certbot with AWS Route53 DNS validation.

## Requirements

- Ansible >= 2.15
- Docker installed on target host
- Target: Ubuntu (focal, jammy, noble) or Debian (bullseye, bookworm)
- AWS credentials with Route53 permissions for DNS-01 challenge

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `certbot_domain` | `example.com` | Domain to obtain certificate for |
| `certbot_config_dir` | `/etc/certbot` | Directory for certbot configuration |
| `certbot_letsencrypt_config_dir` | `/etc/letsencrypt` | Let's Encrypt configuration directory |
| `certbot_service_name` | `certbot` | Name for systemd service/timer |
| `certbot_user` | `certbot` | System user to run certbot |
| `certbot_image_version` | `v3.2.0` | certbot/dns-route53 Docker image version |
| `certbot_cert_name` | `{{ certbot_domain }}` | Certificate name |
| `certbot_deploy_hook_script` | `files/deploy-hook.sh` | Path to deploy hook script (optional) |
| `certbot_aws_access_key` | **required** | AWS access key for Route53 |
| `certbot_aws_secret_access_key` | **required** | AWS secret access key for Route53 |

## Usage

### Install via requirements.yml

```yaml
- src: git@github.com:iamrobertyoung/homelab-ansible-role-certbot.git
  scm: git
  version: main
  name: certbot
```

```bash
ansible-galaxy install -r requirements.yml
```

### Example Playbook

```yaml
- hosts: servers
  roles:
    - role: certbot
      vars:
        certbot_domain: "*.example.com"
        certbot_aws_access_key: "{{ vault_aws_access_key }}"
        certbot_aws_secret_access_key: "{{ vault_aws_secret_key }}"
```

## What Gets Created

### Systemd Units
- `certbot.service` - Oneshot service that runs certbot via Docker
- `certbot.timer` - Timer that triggers renewal daily at midnight
- `certbot-deploy-hook.path` - Watches certificate for changes (if deploy hook configured)
- `certbot-deploy-hook.service` - Runs deploy hook script on certificate change

### Configuration Files
- Docker Compose configuration for certbot container
- Let's Encrypt cli.ini with ECC key settings
- AWS credentials environment file

## License

MIT
