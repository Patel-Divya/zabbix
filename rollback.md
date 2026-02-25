
#  Upgrade Happens
* Repo: https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest%2Bubuntu24.04_all.deb
* Method: https://www.skynats.com/blog/upgrade-zabbix-6-4-to-7-ubuntu-24/
*  install **7.4 repo package**
* Enable 7.4 repo
* Stop server → `systemctl stop zabbix-server`
* Upgrade **only Zabbix packages**:

```bash
sudo apt install --only-upgrade \
zabbix-server-mysql \
zabbix-frontend-php \
zabbix-agent \
zabbix-apache-conf \
zabbix-sql-scripts
```

* Start server → database schema upgrade occurs automatically.

---

# Rollback Plan

## Scenario A — 7.4 Packages Installed, DB NOT Upgraded

If **haven’t started the 7.4 server yet**:

1. Remove 7.4 packages:

```bash
sudo apt remove zabbix-server-mysql zabbix-frontend-php zabbix-agent zabbix-apache-conf zabbix-sql-scripts
```

2. Reinstall **6.4 repo package**:

```bash
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
```

3. Install 6.4 packages (with version lock to match previous):

```bash
sudo apt install \
zabbix-server-mysql=1:6.4.21-1+ubuntu22.04 \
zabbix-frontend-php=1:6.4.21-1+ubuntu22.04 \
zabbix-agent=1:6.4.21-1+ubuntu22.04 \
zabbix-apache-conf=1:6.4.21-1+ubuntu22.04 \
zabbix-sql-scripts=1:6.4.21-1+ubuntu22.04
```


---

## Scenario B — 7.4 Server Started, DB Upgraded

If the **database schema was upgraded to 7.4**:

1. Stop Zabbix server:

```bash
sudo systemctl stop zabbix-server
```

2. Remove 7.4 packages:

```bash
sudo apt remove zabbix-server-mysql zabbix-frontend-php zabbix-agent zabbix-apache-conf zabbix-sql-scripts
```

3. Reinstall 6.4 repo:

```bash
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
```

4. Reinstall 6.4 packages (use `--allow-downgrades` if needed):

```bash
sudo apt install --allow-downgrades \
zabbix-server-mysql=1:6.4.21-1+ubuntu22.04 \
zabbix-frontend-php=1:6.4.21-1+ubuntu22.04 \
zabbix-agent=1:6.4.21-1+ubuntu22.04 \
zabbix-apache-conf=1:6.4.21-1+ubuntu22.04 \
zabbix-sql-scripts=1:6.4.21-1+ubuntu22.04
```

5. Restore database from backup:

```bash
mysql -u root -p < zabbix_backup.sql
```

6. Restore configs and web frontend if needed:

```bash
cp -r /etc/zabbix_backup/* /etc/zabbix/
cp -r /opt/zabbix-backup/* /usr/share/
```

7. Start Zabbix server:

```bash
sudo systemctl start zabbix-server
```
