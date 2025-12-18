# 🎮 OpenArena AWS - Multi-Game Browser Arcade

**Deploy a complete browser-based game arcade on AWS with classic games like 2048, PvP Arena, Pac-Man, and Super Mario Bros!**

[![Terraform](https://img.shields.io/badge/terraform-1.5+-blue.svg)](https://www.terraform.io/)
[![Ansible](https://img.shields.io/badge/ansible-2.9+-red.svg)](https://www.ansible.com/)
[![AWS](https://img.shields.io/badge/AWS-EC2-orange.svg)](https://aws.amazon.com/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## ✨ Features

- 🎯 **Multi-Game Arcade** - Beautiful game selection screen with multiple browser games
- 🌐 **Browser-Based** - All games run directly in your browser, no downloads needed
- 🚀 **One-Command Deploy** - Full infrastructure and game deployment with `make deploy`
- 💰 **Cost-Optimized** - ~$11/month with optional features disabled
- 🔒 **Secure** - SSH restricted, encrypted storage, audit logging
- 📊 **Monitoring** - Cost budgets, anomaly detection, billing alarms
- 🎨 **Modern UI** - Responsive game selection interface

## 🎮 Available Games

| Game | Type | Players | Description |
|------|------|---------|-------------|
| **2048** | Puzzle | Single | Classic number merging puzzle game |
| **PvP Arena** | Action | 1-4 | Fast-paced multiplayer arena shooter |
| **Pac-Man** | Arcade | Single | Classic maze game with ghosts and dots |
| **Super Mario Bros** | Platformer | Single | HTML5 remake with all 32 original levels |
| **Snake** | Arcade | Single | Classic snake game - grow longer by eating food |
| **Tetris** | Puzzle | Single | Legendary puzzle game with falling blocks |
| **Pong** | Arcade | Single | The original arcade classic |
| **Flappy Bird** | Arcade | Single | Navigate through pipes in this addictive game |
| **Space Invaders** | Arcade | Single | Defend Earth from alien invaders |
| **Breakout** | Arcade | Single | Break all the bricks with your paddle |
| **Asteroids** | Arcade | Single | Destroy asteroids in space |
| **Tic-Tac-Toe** | Puzzle | 2 Players | Classic strategy game - get three in a row |

## 🚀 Quick Start

### Prerequisites

- **AWS Account** with billing enabled
- **Terraform** >= 1.5.0 ([Install](https://developer.hashicorp.com/terraform/downloads))
- **Ansible** >= 2.9 ([Install](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html))
- **AWS CLI** configured ([Setup](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html))
- **Git** for cloning the repository

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/openarena-aws.git
cd openarena-aws
```

### 2. Configure Environment

**Create `.env` file:**
```bash
cp .env.example .env
# Edit .env with your values
```

**Key variables:**
```bash
AWS_REGION="us-west-2"
SSH_KEY_NAME="your-key-name"
SSH_PRIVATE_KEY_FILE="./terraform.pem"
SSH_ALLOWED_CIDR="YOUR_IP/32"  # Get with: curl ifconfig.me

# Cloudflare (optional - set to "dummy" if not using)
CLOUDFLARE_API_TOKEN="your-token-or-dummy"
```

**Create `terraform/terraform.tfvars`:**
```bash
cd terraform
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your values
```

**Critical variables:**
```hcl
aws_region          = "us-west-2"
ssh_key_name        = "your-key-name"
ssh_allowed_cidr    = "YOUR_IP/32"

# S3 Bucket Names (MUST be globally unique - include your AWS Account ID)
log_bucket_name     = "openarena-audit-logs-YOUR_ACCOUNT_ID"
flowlog_bucket_name = "openarena-flowlogs-YOUR_ACCOUNT_ID"
cur_bucket_name     = "openarena-cur-YOUR_ACCOUNT_ID"

# Email for cost alerts (MUST CONFIRM SNS SUBSCRIPTION!)
billing_alert_email = "your-email@example.com"
```

### 3. Set Up IAM Permissions

```bash
# Attach required IAM policies
./scripts/attach-iam-policy.sh
```

This creates and attaches:
- `OpenArenaTerraformMonitoring` - SNS, Budgets, Cost Explorer, CUR
- `OpenArenaTerraformSecurity` - IAM, KMS, GuardDuty
- `OpenArenaTerraformCloudTrail` - CloudTrail management

**Also attach via AWS Console:**
- `AmazonEC2FullAccess`
- `AmazonS3FullAccess`
- `AmazonVPCFullAccess`

### 4. Deploy

```bash
# One-command deployment
make deploy

# OR step-by-step with confirmations
make layered-deploy
```

**Deployment takes ~8-10 minutes** and creates:
- EC2 instance (t2.micro)
- Elastic IP
- Security groups
- S3 buckets for logging
- CloudTrail audit logging
- Cost monitoring (budgets, alarms)
- All browser games deployed

### 5. Access Your Arcade

After deployment, visit:
- **Game Selection:** `http://games.alexflux.com` (or your IP)
- **2048:** `http://games.alexflux.com/2048/`
- **PvP Arena:** `http://games.alexflux.com/pvp/`
- **Pac-Man:** `http://games.alexflux.com/pacman/`
- **Super Mario Bros:** `http://games.alexflux.com/mario/`

## 📁 Project Structure

```
openarena-aws/
├── terraform/                 # Infrastructure as Code
│   ├── modules/
│   │   ├── openarena/         # EC2, EIP, Security Groups
│   │   └── cost/              # Monitoring, budgets, CloudTrail
│   ├── main.tf
│   └── terraform.tfvars      # Your config (gitignored)
├── ansible/                   # Configuration management
│   ├── playbooks/
│   │   └── site.yml           # Main playbook
│   ├── roles/
│   │   ├── openarena/         # OpenArena game server
│   │   └── web-game/          # Browser games deployment
│   │       ├── tasks/
│   │       │   ├── deploy-2048.yml
│   │       │   ├── deploy-pvp.yml
│   │       │   ├── deploy-pacman.yml
│   │       │   └── deploy-mario.yml
│   │       └── templates/
│   │           └── game-selection.html.j2
│   └── inventory/
│       └── hosts.ini          # Generated by deploy script
├── scripts/                   # Deployment automation
│   ├── deploy.sh              # Full deployment
│   ├── layered-deploy.sh      # Step-by-step
│   └── destroy.sh             # Teardown
├── Makefile                   # Convenient commands
└── README.md                  # This file
```

## 🛠️ Available Commands

| Command | Description |
|---------|-------------|
| `make deploy` | Full deployment (Terraform + Ansible) |
| `make layered-deploy` | Step-by-step with confirmations |
| `make dry-run` | Validate without deploying |
| `make redeploy` | Tear down and rebuild |
| `make destroy` | Remove all infrastructure |
| `make validate` | Comprehensive validation |
| `make security-scan` | Security-focused scan |
| `make quick-check` | Fast syntax validation |

## 💰 Cost Breakdown

### Monthly Cost (Cost-Optimized Configuration)

| Service | Cost | Notes |
|---------|------|-------|
| EC2 t2.micro | $8.50 | Free tier eligible (750 hrs/month) |
| EBS Storage (8 GB) | $0.80 | GP2 volume |
| Data Transfer | $0.90 | Outbound traffic |
| CloudTrail | **FREE** | First trail is free |
| SNS | **FREE** | First 1,000 emails free |
| AWS Budgets | **FREE** | First 2 budgets free |
| **Total** | **~$11/month** | |

### Optional Features (Disabled by Default)

| Feature | Cost | Enable in terraform.tfvars |
|---------|------|---------------------------|
| GuardDuty | $10/month | `enable_guardduty = true` |
| VPC Flow Logs | $3/month | `enable_vpc_flow_logs = true` |
| CloudWatch Logs | $1.50/month | `enable_cloudwatch_logs = true` |

**Total with all features: ~$25/month**

### Cost Optimization Tips

1. **Stop server when not playing:**
   ```bash
   aws ec2 stop-instances --instance-ids $(terraform output -raw instance_id)
   # Saves ~$4-7/month if stopped 50% of the time
   ```

2. **Use free tier:** EC2 t2.micro is free tier eligible (750 hours/month)

3. **Disable optional features:** GuardDuty, VPC Flow Logs disabled by default

## 🔒 Security Features

- ✅ **SSH Access Restricted** - Only your IP can SSH (`/32` CIDR)
- ✅ **Encrypted Storage** - All S3 buckets encrypted (AES-256)
- ✅ **Audit Logging** - CloudTrail enabled (FREE)
- ✅ **Cost Monitoring** - Budgets, anomaly detection, billing alarms
- ✅ **No Hardcoded Secrets** - All credentials in `.env` (gitignored)
- ✅ **Minimal IAM Permissions** - Granular policies per service

### Security Checklist

- [ ] Update `ssh_allowed_cidr` to your actual public IP
- [ ] Never commit `.env`, `terraform.tfvars`, or `.pem` files
- [ ] Confirm SNS subscription emails for cost alerts
- [ ] Review IAM policies before attaching
- [ ] Enable CloudTrail (already enabled by default)

## 🎯 Adding New Games

Want to add more games? It's easy!

### 1. Create Deployment Task

Create `ansible/roles/web-game/tasks/deploy-[gamename].yml`:

```yaml
---
# Deploy [Game Name] to /[gamename]/ subdirectory

- name: Create [Game Name] game directory
  ansible.builtin.file:
    path: "{{ web_game_dir }}/[gamename]"
    state: directory
    mode: "0755"
    owner: "{{ nginx_user }}"
    group: "{{ nginx_user }}"

- name: Download [Game Name] game
  ansible.builtin.get_url:
    url: https://github.com/user/repo/archive/refs/heads/master.zip
    dest: /tmp/[gamename]-master.zip
    mode: "0644"

- name: Extract [Game Name] game
  ansible.builtin.unarchive:
    src: /tmp/[gamename]-master.zip
    dest: /tmp
    remote_src: true
    creates: /tmp/[gamename]-master

- name: Copy [Game Name] game files
  ansible.builtin.shell: |
    if [ -d /tmp/[gamename]-master ]; then
      find /tmp/[gamename]-master -mindepth 1 -maxdepth 1 -exec cp -r {} {{ web_game_dir }}/[gamename]/ \;
      chown -R {{ nginx_user }}:{{ nginx_user }} {{ web_game_dir }}/[gamename]/
    fi
  changed_when: true
```

### 2. Add to Main Deployment

Edit `ansible/roles/web-game/tasks/main.yml`:

```yaml
- name: Deploy [Game Name] game
  ansible.builtin.include_tasks: deploy-[gamename].yml
```

### 3. Add Game Card

Edit `ansible/roles/web-game/templates/game-selection.html.j2`:

```html
<a href="/[gamename]/" class="game-card">
    <span class="game-icon">🎮</span>
    <h2 class="game-title">[Game Name]</h2>
    <span class="badge singleplayer">Single Player</span>
    <span class="badge action">Action</span>
    <p class="game-description">
        [Game description]
    </p>
    <ul class="game-features">
        <li>Feature 1</li>
        <li>Feature 2</li>
    </ul>
    <span class="play-btn">Play Now →</span>
</a>
```

### 4. Add Nginx Route

Edit `ansible/roles/web-game/templates/nginx.conf.j2`:

```nginx
location /[gamename]/ {
    alias {{ web_game_dir }}/[gamename]/;
    try_files $uri $uri/ /[gamename]/index.html;
}
```

### 5. Deploy

```bash
make deploy
```

## 🎮 Suggested Games to Add

Here are some great open-source browser games you can easily add:

| Game | GitHub Repo | Type | Notes |
|------|-------------|------|-------|
| **Snake** | [gabrielecirulli/2048](https://github.com/gabrielecirulli/2048) | Puzzle | Classic snake game |
| **Tetris** | [vladimirgamalyan/tetris](https://github.com/vladimirgamalyan/tetris) | Puzzle | Classic Tetris |
| **Pong** | [freeCodeCamp/Pong](https://github.com/freeCodeCamp/Pong) | Arcade | Classic Pong |
| **Flappy Bird** | [nebez/floppybird](https://github.com/nebez/floppybird) | Arcade | Flappy Bird clone |
| **Space Invaders** | [lmatteis/space-invaders](https://github.com/lmatteis/space-invaders) | Arcade | Classic Space Invaders |
| **Breakout** | [freeCodeCamp/Breakout](https://github.com/freeCodeCamp/Breakout) | Arcade | Classic Breakout |
| **Asteroids** | [freeCodeCamp/Asteroids](https://github.com/freeCodeCamp/Asteroids) | Arcade | Classic Asteroids |

**Requirements for games:**
- ✅ Pure HTML5/CSS/JavaScript (no server-side code)
- ✅ Available on GitHub as zip download
- ✅ No build process required (or pre-built)
- ✅ Works in modern browsers

## 🤝 Contributing

We welcome contributions! Here's how to get started:

### 1. Fork and Clone

```bash
git clone https://github.com/yourusername/openarena-aws.git
cd openarena-aws
```

### 2. Create a Feature Branch

```bash
git checkout -b feature/add-new-game
```

### 3. Make Your Changes

- Add new games (see [Adding New Games](#-adding-new-games))
- Fix bugs
- Improve documentation
- Add features

### 4. Test Your Changes

```bash
# Validate syntax
make quick-check

# Comprehensive validation
make validate

# Security scan
make security-scan
```

### 5. Submit a Pull Request

- Write a clear description of your changes
- Reference any related issues
- Ensure all tests pass
- Update documentation if needed

### Contribution Guidelines

- **Code Style:** Follow existing Ansible/Terraform conventions
- **Documentation:** Update README.md for new features
- **Testing:** Test locally before submitting PR
- **Commits:** Write clear, descriptive commit messages

## 🐛 Troubleshooting

### Common Issues

**1. Pac-Man/Mario deployment fails**
- **Solution:** Already fixed! The deployment now auto-detects directory names.

**2. Terraform init fails - Cloudflare provider error**
- **Solution:** Create `.env` with `CLOUDFLARE_API_TOKEN="dummy"` even if not using Cloudflare.

**3. Ansible fails - Python version error**
- **Solution:** Already fixed! Python 3.8 is auto-installed via EC2 user_data.

**4. SSH connection refused**
- **Solution:** 
  - Verify SSH key path in `.env`
  - Check key permissions: `chmod 400 terraform.pem`
  - Ensure `ssh_allowed_cidr` matches your IP

**5. Games not loading**
- **Solution:**
  - Check Nginx is running: `sudo systemctl status nginx`
  - Verify files exist: `ls -la /var/www/html/[gamename]/`
  - Check Nginx logs: `sudo tail -f /var/log/nginx/error.log`

### Debug Commands

```bash
# Check Terraform state
cd terraform && terraform show

# SSH to server
ssh -i terraform.pem ec2-user@$(terraform output -raw public_ip)

# Check game server status
sudo systemctl status nginx
sudo systemctl status openarena

# View logs
sudo journalctl -u nginx -f
sudo journalctl -u openarena -f

# Test game URLs
curl http://localhost/2048/
curl http://localhost/pvp/
```

## 📚 Additional Documentation

- **[DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)** - Detailed deployment instructions
- **[terraform/SECURITY.md](terraform/SECURITY.md)** - Security best practices
- **[SECURITY_AUDIT.md](SECURITY_AUDIT.md)** - Security audit report

## 📝 License

This project is for educational and personal use.

## 🙏 Acknowledgments

- **OpenArena** - Free, open-source Quake III Arena clone
- **2048** - [gabrielecirulli/2048](https://github.com/gabrielecirulli/2048)
- **PvP** - [kesiev/pvp](https://github.com/kesiev/pvp)
- **Pac-Man** - [GerardAlbajar/Pacman-js](https://github.com/GerardAlbajar/Pacman-js)
- **Super Mario Bros** - [umaim/Mario](https://github.com/umaim/Mario)
- **Terraform** - Infrastructure as Code
- **Ansible** - Configuration management

## 📞 Support

- **Issues:** [GitHub Issues](https://github.com/yourusername/openarena-aws/issues)
- **Discussions:** [GitHub Discussions](https://github.com/yourusername/openarena-aws/discussions)

---

**Made with ❤️ for the gaming community**

**Status:** ✅ Production Ready | **Last Updated:** 2025-12-18
