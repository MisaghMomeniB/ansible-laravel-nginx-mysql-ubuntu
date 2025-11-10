# Ansible Laravel Stack on Ubuntu (Nginx + PHP-FPM latest + MySQL)

Provision an Ubuntu server for a **fresh Laravel app** with **Nginx**, **latest PHP-FPM** (Ondřej PPA), **MySQL**, **Composer**, and sensible defaults — all **idempotent** via Ansible.

> One command from zero to running at `http://SERVER_IP/`.

---

## ✨ What you get

- Latest **PHP-FPM** (via `ppa:ondrej/php`) with common extensions
- **Nginx** vhost pointing to `public/`
- **MySQL** server + app **database** + **user** with least privileges
- **Composer** global install
- Fresh **Laravel** under `/var/www/laravel`
- Proper **permissions** for `storage/` and `bootstrap/cache`
- **Systemd** services enabled (nginx, mysql, php-fpm)
- Idempotent: safe to re-run

---

## 🧱 Prerequisites

- Control machine with **Ansible 2.15+** and Python 3
- SSH access to Ubuntu target (22.04/24.04 tested)
- Install required collection:
  ```bash
  ansible-galaxy collection install -r requirements.yml
  ```

---

## 🚀 Quick Start

1. **Clone** the repo:
   ```bash
   git clone https://github.com/your-org/ansible-laravel-nginx-mysql-ubuntu.git
   cd ansible-laravel-nginx-mysql-ubuntu
   ```

2. **Inventory**: edit `inventory.ini`
   ```ini
   [web]
   your_server_ip ansible_user=ubuntu
   ```

3. **Variables**: update `group_vars/all.yml` (at least `db_pass` and `server_name`).

4. **Run**:
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ansible-playbook site.yml -K
   ```

5. Browse to:
   ```
   http://your_server_ip/
   ```
   (If you set a domain in `server_name`, point DNS to the server.)

---

## 🔧 Customization

- **Laravel path**: change `app_dir` (default `/var/www/laravel`)
- **Domain**: set `server_name` (e.g. `example.com`)
- **DB settings**: `db_name`, `db_user`, `db_pass`
- **PHP extensions**: edit the list in `site.yml` (task: *Install PHP-FPM and common extensions*)
- **Firewall**: UFW allow for Nginx is included as an optional non-failing shell task

> The playbook auto-detects PHP minor version and wires Nginx → PHP-FPM socket accordingly.

---

## 🔐 Security Notes

- **Change `db_pass`** before running.
- Consider enabling **UFW** and restricting SSH.
- For production, add **TLS** (Certbot). Example (manual):
  ```bash
  sudo apt-get install -y certbot python3-certbot-nginx
  sudo certbot --nginx -d example.com
  ```

---

## 🧪 Idempotency & Re-runs

It’s safe to run `ansible-playbook site.yml` multiple times. Tasks use:
- `creates:` guards (Composer, Laravel scaffold)
- Proper `state: present` for packages/db
- Handlers to reload Nginx only when config changes

---

## ❓ Troubleshooting

- **Composer rate limits**: retry; the task is guarded with `creates:` so it won’t re-install unnecessarily.
- **MySQL auth**: we use UNIX socket & implicit admin. On hardened hosts, you may need to set a root password or create an admin user and pass `login_user`/`login_password` to `community.mysql` tasks.
- **PHP-FPM service name**: auto-detected via `php -r 'echo PHP_MAJOR_VERSION."."PHP_MINOR_VERSION;'`. If customized, adjust the service name and socket path in the Nginx config task.

---

## 🗺️ Files

```
site.yml                 # Main playbook
inventory.ini            # Sample inventory
group_vars/all.yml       # Tweak app/db/php settings here
requirements.yml         # Ansible Galaxy dependencies
ansible.cfg              # Local Ansible settings
README.md                # This file
LICENSE                  # MIT (optional)
```

---

## 📄 License

MIT — do whatever you want, just keep the notice.
