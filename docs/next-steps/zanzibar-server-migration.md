# Zanzibar: Moving HCPI to a New Server

This is the plan for moving Zanzibar's HCPI from its current server to a new one, with no data loss and as little downtime as possible. It describes **what** we do and in **what order**. For the individual commands it points to the existing guides:

- [Exporting HCPI from a Linux Server](../extraction/linux-export.md): getting the code, database and filestore off the old server
- [Linux Server Installation](../installation/linux.md): building the new server and restoring onto it
- [HTTPS with Nginx](https-nginx.md) / [HTTPS with Apache](https-apache.md): the public web address and certificate

!!! abstract "The plan in one paragraph"
    Build the new server and do a **trial migration** onto it while the old server keeps running normally. Once the trial passes, pick a cutover window. In that window the phones sync everything, the old server is stopped, a **final** copy is taken and restored on the new server, and the domain name is pointed at the new server. The old server is left switched off but intact for a few weeks, so we can go back to it if something goes wrong.

## 0. Facts to collect first

Fill this table in before anything else. Every later step depends on it.

| Item | Old server | New server |
|---|---|---|
| Hostname / IP address | | |
| SSH user and port | | |
| Operating system (`lsb_release -a`) | | |
| PostgreSQL version (`psql --version`) | | |
| Python version | | |
| Odoo version / commit (Export guide, Step 11) | 18.0 / | 18.0 |
| Install folder (`INSTALL`) | e.g. `/opt/hcpi` | |
| Config file path | | |
| Database name (`db_name`) | | |
| User that runs Odoo (`RUN_USER`) | | |
| Filestore location | | |
| Size of database + filestore (`du -sh`) | | free disk: |
| Public web address (domain) | | *(same, see §1)* |
| Who manages DNS for that domain | | |
| Reverse proxy (Nginx / Apache / none) and certificate | | |
| Outgoing mail server configured in Odoo? | | |
| Code branch | `zar_18` | `zar_18` |

!!! warning "PostgreSQL version"
    The new server's PostgreSQL must be the **same or newer** major version than the old server's. A dump from PostgreSQL 16 will not restore into PostgreSQL 14.

## 1. Key decision: keep the same web address

**Recommended: keep the same domain name and only change where it points (DNS).**

The reason is the mobile app. The Zanzibar server address (`ZAR_DOMAIN`) is **built into the Flutter app** ([Mobile App, §5](../understanding-the-codebase/mobile-app.md)). If the address changes, every collector's phone needs a new app build before it can connect again. If the domain stays the same, the phones notice nothing.

If the address *must* change, add these to the plan:

- Build and distribute a new APK with the new `ZAR_DOMAIN` **before** the cutover.
- After the restore, update `web.base.url` in **Settings → Technical → System Parameters** on the new server.
- Tell all web users the new address.

## 2. Roles

| Role | Who | Responsibilities |
|---|---|---|
| Zanzibar IT | *(name)* | Server access, new server provisioning, DNS, firewall |
| HCPI support | *(name)* | Export / restore, configuration, verification |
| HCPI lead / supervisors | *(name)* | Choose the window, tell users and collectors, sign off on verification |

## 3. Timeline at a glance

| When | Phase | Old server |
|---|---|---|
| T-14 days | Prepare the new server | Running normally |
| T-7 days | Trial migration and testing | Running normally |
| T-2 days | Lower DNS TTL; announce the window | Running normally |
| **T-0** | **Cutover** (allow ~half a day) | Stopped |
| T+1 to T+14 | Aftercare, daily checks | Off but kept intact |
| T+30 | Decommission old server | Wiped |

Pick a cutover day **outside the active price-collection period** and away from index-computation deadlines.

## Phase 1: Prepare the new server (T-14)

Old server: untouched.

1. **Provision** the new machine (Ubuntu LTS recommended), with enough disk for at least **3×** the database and filestore combined (the data, the export files, and growth).
2. **Install system packages, PostgreSQL, and create the DB user**: [Linux install](../installation/linux.md), Steps 1–4.
3. **Clone Odoo 18.0 and create the Python venv**: Linux install, Steps 5–6. Use the **same Python minor version** as the old server if possible.
4. **Firewall**: allow SSH, 80 and 443 only. Odoo's port (9201) should not be open to the public once the reverse proxy is in place.
5. **Install the reverse proxy** (Nginx or Apache, matching what the old server uses) but **do not request the certificate yet**. The domain still points at the old server (see Phase 4).
6. **Set up the `hcpi` systemd service**: Linux install, *systemd* section. Keep it **disabled** for now.
7. **Confirm the new server can reach the old one** over SSH (for the server-to-server copy in Phase 4). If it can't, the files go through a laptop instead (Export guide, Step 9).

## Phase 2: Trial migration (T-7)

This is a full rehearsal while the old server keeps running. It shows that the process works and how long each step takes. **Write down the time each step takes**, because that sets the length of the real cutover window.

1. **Export from the old server** while it runs: [Export guide](../extraction/linux-export.md), Steps 1–8. This produces `hcpi.dump`, `hcpi-files.zip` and `hcpi-filestore.zip`.
2. **Copy the three files to the new server** (see Phase 4, step 5 for the command).
3. **Unpack the code and config**: Linux install, Step 5, *Option B*. Then check:
    - `custom/HCPI` is a git checkout of **`zar_18`** (`git -C custom/HCPI branch`). Future updates depend on this ([Updating a Country Deployment](updating-deployments.md)).
    - `hcpi.conf` has the right paths, `db_password`, and `server_wide_modules = base,web,queue_job` ([queue_job](../modules/infrastructure.md)).
4. **Stop the trial server from acting on its own.** In `hcpi.conf`, temporarily set:

    ```ini
    max_cron_threads = 0
    ```

    so scheduled actions (emails, automated jobs) don't run against the copied data and send duplicate emails to real users. Remove this line before the real cutover.

5. **Restore the database and filestore**: Linux install, Step 8, *Option A* (A1 and A2). Put the filestore under the **user the service runs as**.
6. **Start the service** and check the log (`/opt/hcpi/log/hcpi.log`) for errors.
7. **Test it** by browsing to `http://<new-server-ip>:9201` from inside the network (or through an SSH tunnel) and running the [verification checklist](#verification-checklist) below. The mobile-app items can wait until Phase 4.
8. **Fix whatever came up**, update this document, and repeat the trial if any step had to change.

!!! tip "Go / no-go"
    Don't schedule the cutover until a trial has passed the whole checklist without manual fixes.

## Phase 3: Pre-cutover (T-2)

1. **Lower the DNS TTL** for the HCPI domain to 300 seconds (5 minutes). DNS changes then take effect quickly on the day, and a rollback is just as quick.
2. **Announce the window** to all web users and collectors: date, start time, expected duration, and "don't enter data during this window".
3. **Tell collectors to sync their phones before the window starts.** Observations recorded offline stay queued on the phone until they sync (see [Mobile App, §7](../understanding-the-codebase/mobile-app.md)). Anything that reaches the old server *after* the final export is lost.
4. **Confirm access**: SSH to both servers, sudo, DNS control panel, and the Odoo admin login.

## Phase 4: Cutover day (T-0)

**Old server, start of the window:**

1. **Check the phones are synced**: in HCPI, confirm the latest observations from each collection centre have arrived. Follow up with any collector whose data is missing.
2. **Check that no background jobs are running**: **Queue Jobs** menu. Wait until nothing is *Started* or *Enqueued*. A job that was running when the copy was taken stays stuck on the new server.
3. **Stop Odoo on the old server** so nothing new can be written:

    ```bash
    sudo systemctl stop hcpi        # or whatever the service is called
    sudo systemctl disable hcpi     # so a reboot doesn't bring it back
    ```

4. **Take the final export**: [Export guide](../extraction/linux-export.md), Steps 3–8 (same as the trial).

**New server:**

5. **Copy the files directly from the old server** and check they arrived intact:

    ```bash
    mkdir -p ~/migration
    rsync -avP -e "ssh -p <ssh_port>" olduser@<old-server>:/tmp/hcpi-export/ ~/migration/

    # Checksums must match on both servers
    sha256sum ~/migration/*                         # on the new server
    ssh olduser@<old-server> "sha256sum /tmp/hcpi-export/*"   # old server
    ```

6. **Clear out the trial data**: stop the service, drop and recreate the database, and delete the trial filestore (Linux install, Step 8, *"Re-running the restore"*).
7. **Restore the final database and filestore**: Linux install, Step 8, *Option A*.
8. **Remove `max_cron_threads = 0`** from `hcpi.conf`, then **enable and start** the service:

    ```bash
    sudo systemctl enable hcpi
    sudo systemctl start hcpi
    ```

9. **Run the [verification checklist](#verification-checklist)** using the server IP, before any users are pointed at it.

**Switch over:**

10. **Point the DNS record** for the HCPI domain at the new server's IP.
11. **Get the HTTPS certificate** on the new server once DNS resolves to it (`dig +short <domain>` returns the new IP): [Nginx](https-nginx.md) / [Apache](https-apache.md).
12. **Test from outside**: `https://<domain>` loads with a valid padlock, and you can log in.
13. **Test with one phone**: log in from the mobile app, download the assignments, submit one test observation, and confirm it appears on the server.
14. **Announce that HCPI is back.** Raise the DNS TTL back to its normal value.

## Verification checklist

Run this in the trial (Phase 2) and again on the day (Phase 4). Record the results.

**Record counts must match the old server.** Run on **both** servers and compare:

```bash
sudo -u postgres psql -d <db_name> -c "
SELECT 'users' AS what, count(*) FROM res_users
UNION ALL SELECT 'attachments',          count(*) FROM ir_attachment
UNION ALL SELECT 'outlets',              count(*) FROM hcpi_outlet
UNION ALL SELECT 'questionnaires',       count(*) FROM hcpi_data_collection
UNION ALL SELECT 'collection centres',   count(*) FROM zar_collection_center;"
```

(Run it on the old server right after the final export, before it's switched off.)

**Functional checks:**

- [ ] Admin and an ordinary user can log in
- [ ] Outlets list loads, with regions, districts and collection centres filled in
- [ ] A questionnaire with **photos/attachments** opens and the images display (a broken image means a filestore problem)
- [ ] The latest observations from before the cutover are present
- [ ] An **index computation** runs to completion (this proves `queue_job` is working)
- [ ] The dashboard loads and shows the same figures as on the old server for the last published period
- [ ] Reports / exports download
- [ ] **Settings → Technical → System Parameters → `web.base.url`** shows the correct public address
- [ ] Outgoing mail (if used) sends a test email
- [ ] Scheduled actions are active (**Settings → Technical → Scheduled Actions**)
- [ ] *(day only)* HTTPS certificate is valid and HTTP redirects to HTTPS
- [ ] *(day only)* A phone logs in, syncs, and submits an observation

## Rollback plan

The old server is stopped, **not changed**, so we can always go back to it.

| Situation | What to do |
|---|---|
| Checks fail **before** the DNS switch (step 9) | Re-enable and start Odoo on the old server and tell users the move is postponed. No data lost. |
| Serious problem found **soon after** the switch, before real data is entered | Point DNS back to the old server, start Odoo there, and investigate. |
| Problem found **after users have entered data** on the new server | Prefer fixing forward on the new server. Rolling back now would lose the new entries. Only roll back if the lead agrees, and export the new entries first. |

Agree who makes the go/no-go call **before** the day. By default it's the HCPI lead, on advice from HCPI support.

## Phase 5: Aftercare (T+1 to T+30)

1. **Daily checks for the first week**: the log has no new errors, index jobs complete, and phones are syncing.
2. **Set up automatic backups on the new server**: a nightly `pg_dump` and a copy of the filestore, kept **off the server**. Test a restore from it once.
3. **Keep the old server switched off but intact** for at least **2 weeks** (until after the next index publication, if possible).
4. **Decommission** at T+30, once the lead signs off: keep one final archived export, then wipe the old server's disks. Delete the leftover export files (`/tmp/hcpi-export`, `~/migration`) on both servers. They contain the full database and the admin password.
5. **Update the records**: this document, the server table in §0, and anyone who keeps a server list (EAC Secretariat, mobile app maintainers).

## Notes and lessons

*Add anything learned during the trial or cutover here.*

**Trial migration, November 2026:**

- **Check how traffic actually reaches the server before planning a DNS change.** In Zanzibar the public name resolves to the office firewall's public IP. The firewall forwards the Odoo port straight to the server and sends port 80 to a separate IIS reverse proxy. The switch is therefore a change to the **firewall port-forward and the proxy target**, not to DNS. A short `tcpdump` on the Odoo port shows which `Host` headers and proxies are in use.
- **New Ubuntu releases ship a Python Odoo 18 doesn't support** (Ubuntu 26.04 has Python 3.14). Install the old server's exact Python version with [uv](https://docs.astral.sh/uv/) (`uv python install 3.13.5`, then `uv venv --python 3.13.5`) and install the old venv's `pip freeze`. Only `lxml` had to move from 5.2.1 to 5.3.2, because there is no Python 3.13 wheel for 5.2.1.
- A PostgreSQL 14 dump restored into PostgreSQL 18 with no errors (`pg_restore --no-owner --role=<odoo user>`).
- Keep the same service name, user, paths and ports as the old server, so the network change is only an IP swap.
- Export, copy, restore and start took about **2 minutes** for a 1.4 GB database. The `hcpi-export.sh` / `hcpi-import.sh` scripts in the docs repository (`tools/zanzibar-migration/`) do the whole cycle and compare record counts.
- Check that existing backups are real. Look at the file sizes: a few KB means an error page, not a database.
