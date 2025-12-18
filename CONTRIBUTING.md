# Contributing to OpenArena AWS

Thank you for your interest in contributing! This document provides guidelines and instructions for contributing to the project.

## 🎯 How to Contribute

### Reporting Bugs

If you find a bug, please open an issue with:
- **Clear title** describing the issue
- **Steps to reproduce** the bug
- **Expected behavior** vs **actual behavior**
- **Environment details** (OS, Terraform version, Ansible version)
- **Error messages** or logs (if applicable)

### Suggesting Features

We welcome feature suggestions! Open an issue with:
- **Clear description** of the feature
- **Use case** - why would this be useful?
- **Proposed implementation** (if you have ideas)

### Adding Games

The easiest way to contribute is adding new browser games! See the [Adding New Games](#adding-new-games) section below.

### Code Contributions

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/your-feature-name`
3. **Make your changes**
4. **Test thoroughly** (see [Testing](#testing))
5. **Commit with clear messages** (see [Commit Messages](#commit-messages))
6. **Push to your fork**: `git push origin feature/your-feature-name`
7. **Open a Pull Request**

## 🎮 Adding New Games

### Requirements

Games must be:
- ✅ **Pure HTML5/CSS/JavaScript** - No server-side code required
- ✅ **Available on GitHub** - As a zip download or git repository
- ✅ **No build process** - Or pre-built files available
- ✅ **Browser-compatible** - Works in modern browsers (Chrome, Firefox, Safari, Edge)
- ✅ **Open source** - With a permissive license

### Step-by-Step Guide

#### 1. Create Deployment Task

Create `ansible/roles/web-game/tasks/deploy-[gamename].yml`:

```yaml
---
# Deploy [Game Name] to /[gamename]/ subdirectory
# Brief description of the game

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
  register: game_download

- name: Extract [Game Name] game
  ansible.builtin.unarchive:
    src: /tmp/[gamename]-master.zip
    dest: /tmp
    remote_src: true
    creates: /tmp/[gamename]-master

- name: Find extracted [Game Name] directory
  ansible.builtin.shell: |
    find /tmp -maxdepth 1 -type d -name "*[gamename]*" | head -1
  register: [gamename]_dir
  changed_when: false

- name: Copy [Game Name] game files to game directory
  ansible.builtin.shell: |
    if [ -n "{{ [gamename]_dir.stdout }}" ] && [ -d "{{ [gamename]_dir.stdout }}" ]; then
      find {{ [gamename]_dir.stdout }} -mindepth 1 -maxdepth 1 -exec cp -r {} {{ web_game_dir }}/[gamename]/ \;
      chown -R {{ nginx_user }}:{{ nginx_user }} {{ web_game_dir }}/[gamename]/
      echo "[Game Name] files copied successfully"
    else
      echo "Error: [Game Name] directory not found"
      exit 1
    fi
  changed_when: true
```

**Tips:**
- Use lowercase for directory names (e.g., `snake`, `tetris`)
- Handle different directory names after extraction (use `find` to detect)
- Set proper ownership with `chown`

#### 2. Add to Main Deployment

Edit `ansible/roles/web-game/tasks/main.yml`:

```yaml
- name: Deploy [Game Name] game
  ansible.builtin.include_tasks: deploy-[gamename].yml
```

Add it after the other game deployments.

#### 3. Add Game Card to Selection Screen

Edit `ansible/roles/web-game/templates/game-selection.html.j2`:

Add a new game card in the `.games-grid` section:

```html
<a href="/[gamename]/" class="game-card">
    <span class="game-icon">🎮</span>
    <h2 class="game-title">[Game Name]</h2>
    <span class="badge singleplayer">Single Player</span>
    <span class="badge puzzle">Puzzle</span>
    <p class="game-description">
        Brief description of the game and what makes it fun!
    </p>
    <ul class="game-features">
        <li>Feature 1</li>
        <li>Feature 2</li>
        <li>Feature 3</li>
    </ul>
    <span class="play-btn">Play Now →</span>
</a>
```

**Badge options:**
- `singleplayer` - Blue badge
- `multiplayer` - Green badge
- `puzzle` - Orange badge
- `action` - Red badge
- `arcade` - Purple badge
- `platformer` - Pink badge

**Icon suggestions:**
- 🎮 General game
- 🔢 Numbers/puzzle
- ⚔️ Action/shooter
- 👻 Arcade
- 🍄 Platformer
- 🐍 Snake
- 🧩 Puzzle
- 🎯 Target/arcade

#### 4. Add Nginx Route

Edit `ansible/roles/web-game/templates/nginx.conf.j2`:

Add a location block:

```nginx
location /[gamename]/ {
    alias {{ web_game_dir }}/[gamename]/;
    try_files $uri $uri/ /[gamename]/index.html;
}
```

#### 5. Update README

Edit `README.md`:

1. Add to the "Available Games" table
2. Add to the "Suggested Games to Add" section (if applicable)

#### 6. Test Your Changes

```bash
# Validate syntax
make quick-check

# Test deployment (use dry-run first)
make dry-run

# Deploy to test server
make deploy
```

#### 7. Submit Pull Request

- Write a clear description
- Include screenshots if possible
- Reference the game's GitHub repository
- Test that the game works in multiple browsers

## 🧪 Testing

### Pre-Submission Checklist

- [ ] **Syntax validation passes**: `make quick-check`
- [ ] **Comprehensive validation passes**: `make validate`
- [ ] **Security scan passes**: `make security-scan`
- [ ] **Game works** in Chrome, Firefox, Safari, Edge
- [ ] **Game loads** from the selection screen
- [ ] **No hardcoded secrets** or credentials
- [ ] **Documentation updated** (README.md)
- [ ] **Code follows** existing style conventions

### Local Testing

```bash
# Quick syntax check
make quick-check

# Full validation
make validate

# Security scan
make security-scan

# Dry-run deployment (no actual deployment)
make dry-run
```

### Testing Game Deployment

1. **Deploy to test server:**
   ```bash
   make deploy
   ```

2. **Verify game loads:**
   ```bash
   curl http://your-server/[gamename]/
   ```

3. **Check files exist:**
   ```bash
   ssh -i terraform.pem ec2-user@your-server
   ls -la /var/www/html/[gamename]/
   ```

4. **Test in browser:**
   - Visit `http://your-server/[gamename]/`
   - Verify game plays correctly
   - Test on multiple browsers

## 📝 Commit Messages

Write clear, descriptive commit messages:

**Good:**
```
Add Snake game to arcade

- Created deploy-snake.yml task
- Added game card to selection screen
- Configured Nginx route for /snake/
- Updated README with game information
```

**Bad:**
```
fix stuff
```

**Format:**
```
Short summary (50 chars or less)

More detailed explanation if needed. Wrap at 72 characters.
Explain what and why, not how.

- Bullet points for multiple changes
- Reference issues: Fixes #123
```

## 🎨 Code Style

### Ansible

- Use `ansible.builtin` modules when possible
- Use `changed_when` for idempotency
- Add comments for complex logic
- Use descriptive variable names

### Terraform

- Use consistent formatting: `terraform fmt`
- Use descriptive resource names
- Add comments for complex configurations
- Follow existing module structure

### YAML

- Use 2-space indentation
- Quote strings with special characters
- Use lists for multiple items
- Keep line length under 120 characters

## 📚 Documentation

### When to Update Documentation

- ✅ Adding new games
- ✅ Changing deployment process
- ✅ Adding new features
- ✅ Fixing bugs (update troubleshooting)
- ✅ Changing configuration options

### Documentation Files

- **README.md** - Main documentation
- **CONTRIBUTING.md** - This file
- **DEPLOYMENT_GUIDE.md** - Detailed deployment guide
- **terraform/SECURITY.md** - Security best practices

## 🐛 Bug Fixes

### Before Fixing

1. **Reproduce the bug** locally
2. **Identify root cause**
3. **Check for existing issues** or PRs
4. **Create issue** if none exists

### Fixing

1. **Create branch**: `git checkout -b fix/bug-description`
2. **Write fix** with tests if applicable
3. **Test thoroughly**
4. **Update documentation** if needed
5. **Submit PR** with clear description

## 💡 Feature Ideas

Some ideas for contributions:

### Games
- Snake
- Tetris
- Pong
- Flappy Bird
- Space Invaders
- Breakout
- Asteroids
- Chess
- Checkers

### Features
- Game statistics tracking
- High score leaderboard
- User authentication
- Custom game themes
- Mobile-responsive improvements
- Game search/filter
- Favorites system
- Share game links

### Infrastructure
- Multi-region deployment
- Auto-scaling
- CDN integration
- SSL/TLS certificates
- Monitoring dashboards
- Backup automation

## ❓ Questions?

- **Open an issue** for questions
- **Check existing issues** first
- **Join discussions** on GitHub Discussions

## 🙏 Thank You!

Your contributions make this project better for everyone. Thank you for taking the time to contribute!

---

**Happy Contributing!** 🎮

