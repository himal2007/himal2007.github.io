---
layout: distill
title: OneDrive on Ubuntu - Syncing files with ease for multiple accounts
description: A comprehensive guide to setting up OneDrive synchronization on Ubuntu using abraunegg's onedrive tool, with special focus on managing multiple accounts.
tags: ubuntu, onedrive, linux, file-sync, microsoft-365
categories: linux, productivity
giscus_comments: true
date: 2025-05-21
featured: true

authors:
  - name: Himal Shrestha
    url: "https://himal2007.github.io"
    affiliations:
      name: MDU-PHL, The University of Melbourne

toc:
  - name: OneDrive on Ubuntu - Syncing files with ease for multiple accounts
  - name: Getting Started with onedrive
  - name: Managing Multiple Accounts
    subsections:
      - name: Step-by-step - Setting Up Multiple Accounts
      - name: Automating Sync with systemd
  - name: Tips and Tricks
    subsections:
      - name: Syncing smarter, not harder
      - name: Using --monitor mode with caution
      - name: The Great --resync Caveat
  - name: Troubleshooting Common Issues
  - name: Final Thoughts
---

# OneDrive on Ubuntu: Syncing files with ease for multiple accounts

I recently made the full switch to Ubuntu—no dual-boot, no fallback to Windows, just Ubuntu all the way. It has been a great journey so far, but it comes with its own set of challenges. I love the control and freedom Linux offers, but I quickly hit a snag when it came to integrating my day-to-day tools from work. One of the biggest hurdles? File management via OneDrive. At work, Office 365 is central to collaboration and documentation, and OneDrive is the glue that holds my file management together. Ubuntu, for all its power, doesn’t get much love from Microsoft, and I found myself missing that seamless file sync I once took for granted. That’s when I started hunting for solutions and stumbled upon a gem: [onedrive](https://github.com/abraunegg/onedrive) by [abraunegg](https://github.com/abraunegg).

It's a robust and actively maintained command-line tool for syncing OneDrive with Linux. I've been using it for a few months now with both my work and personal accounts, and after a bit of trial and error, I've fine-tuned the experience for smooth, reliable syncing. Here's how you can do the same.

---

## TLDR: Setting Up OneDrive on Ubuntu

- **Install**: Compile `onedrive` from source following [GitHub instructions](https://github.com/abraunegg/onedrive/blob/master/docs/INSTALL.md)
- **Basic usage**: `onedrive --sync` (for a single account)
- **Multiple accounts**: Create separate config directories for each account
  ```bash
  mkdir -p ~/.config/onedrive-personal ~/.config/onedrive-work
  mkdir -p ~/onedrive-personal ~/onedrive-work
  ```
- **Sync command**: `onedrive --confdir="~/.config/onedrive-personal" --sync`
- **Selective sync**: Create a `sync_list` file to include/exclude specific folders
- **Caution**: Be careful with `--monitor` mode and `--resync` (can create conflict files)
- **Best practice**: Avoid running automatic sync on multiple machines for the same account

For detailed setup instructions and troubleshooting tips, read on!

---

## Getting Started with `onedrive`

The first step is installing the tool. Depending on your Ubuntu version, you can either use a prebuilt package or compile it from source.

For the latest features and better reliability, I recommend compiling from source following the instructions on the [GitHub page](https://github.com/abraunegg/onedrive/blob/master/docs/INSTALL.md).

<details>
<summary><strong>Before You Begin: Resolving libcurl and HTTPS Issues</strong></summary>

Some Ubuntu systems may encounter issues with HTTPS due to outdated `libcurl` versions. To avoid this, it's recommended to install the latest `curl` from source:

```bash
sudo apt remove curl
sudo apt install -y libssl-dev
cd /tmp
curl -O https://curl.se/download/curl-8.7.1.tar.gz
 tar -xvzf curl-8.7.1.tar.gz
cd curl-8.7.1
./configure --with-ssl
make -j$(nproc)
sudo make install
```

You can verify the version:

```bash
curl --version
```

Make sure `/usr/local/bin` is in your PATH so that the newly installed version is used.

</details>


Once installed, you can authorize your OneDrive account and start syncing with a simple command:

```bash
onedrive --sync
```

## Managing Multiple Accounts

To manage multiple OneDrive accounts (e.g., work and personal), you need to dive a bit deeper into the configuration. The beauty of `onedrive` is its flexibility, allowing you to create separate configuration directories and thus, allowing you to sync files from multiple accounts seamlessly. Here's how you can set it up, step by step:

1. **Create config directories for each account**:

```bash
mkdir -p ~/.config/onedrive-personal
mkdir -p ~/.config/onedrive-work
```

2. **Create directories for syncing**:

```bash
mkdir -p ~/onedrive-personal
mkdir -p ~/onedrive-work
```

3. **Download the default config file**:

```bash
wget https://raw.githubusercontent.com/abraunegg/onedrive/master/config -O ~/.config/onedrive-personal/config
wget https://raw.githubusercontent.com/abraunegg/onedrive/master/config -O ~/.config/onedrive-work/config
```

4. **Edit the config files**:
```sh
# For personal account
nano ~/.config/onedrive-personal/config
# add "~/onedrive-personal" to the sync_dir line
# For work account
nano ~/.config/onedrive-work/config
# add "~/onedrive-work" to the sync_dir line
```

5. **Check the configuration**:

Check the configuration for each account to ensure everything is set up correctly (most importantly, the `sync_dir`):

```bash
onedrive --confdir="~/.config/onedrive-personal" --display-config
onedrive --confdir="~/.config/onedrive-work" --display-config
```
If everything looks good, you can proceed to the next step.

6. **Authenticate each account**:

```bash
onedrive --confdir="~/.config/onedrive-personal"
onedrive --confdir="~/.config/onedrive-work"
```
This will prompt you to authenticate via a URL. Follow the link and paste the response back in the terminal.

7. **Synchronize each account**:

```bash
onedrive --confdir="~/.config/onedrive-personal" --sync
onedrive --confdir="~/.config/onedrive-work" --sync
```

This will start the synchronization process for each account. Depending on the number and size of files, this may take some time.

Once the sync is complete, you would like to monitor the sync process. You can do this by running:

```bash
onedrive --confdir="~/.config/onedrive-personal" --monitor
onedrive --confdir="~/.config/onedrive-work" --monitor
```

Or, you can automate the sync process using `systemd` services.

### Automating Sync with `systemd`

<details>
<summary>⚠️ A Word of Caution About Automating Sync</summary>

While it might be tempting to automate the sync process using systemd services, I’d urge you to think twice—especially if you’re working across multiple machines.

I learned this the hard way: I had set up `onedrive` to auto-sync on more than one of my computers, and it led to a nightmare of conflicts. Thousands of `.safebackup` files were suddenly being created, cluttering both my local folders and cloud storage. What was meant to save time actually caused hours of cleanup.

Automating sync is absolutely fine if you’re sticking to a single machine. But once you introduce multiple systems connecting to the same OneDrive account, the risk of conflicts rises significantly. If you’re in that boat (like I am), manual syncing might just save you a headache.

</details>



To automate the sync process, you can create `systemd` services for each account. This way, you can have them run in the background and monitor changes automatically. To create a `systemd` service for each account, follow these steps:

1. **Create a systemd service file for each account**:

For the personal account:
```bash
cat <<EOF > ~/.config/systemd/user/onedrive-personal.service
[Unit]
Description=OneDrive Personal Sync
After=network.target
[Service]
ExecStart=/usr/local/bin/onedrive --monitor --confdir=/home/$(whoami)/.config/onedrive-personal
Restart=on-failure
[Install]
WantedBy=default.target
EOF
```

For the work account:
```bash
cat <<EOF > ~/.config/systemd/user/onedrive-work.service
[Unit]
Description=OneDrive Work Sync
After=network.target
[Service]
ExecStart=/usr/local/bin/onedrive --monitor --confdir=/home/$(whoami)/.config/onedrive-work
Restart=on-failure
[Install]
WantedBy=default.target
EOF
```
Replace `/usr/local/bin/onedrive` with the actual path to your `onedrive` binary if it's different.
2. **Reload the systemd user daemon**:

```bash
systemctl --user daemon-reload
```
3. **Enable and start the services**:

```bash
systemctl --user enable onedrive-personal.service --now
systemctl --user enable onedrive-work.service --now
```
This will start the sync services for both accounts and ensure they run automatically on boot.

You can check the status of the services with:
```bash
systemctl --user status onedrive-personal.service
systemctl --user status onedrive-work.service
```

Congratulations! You now have a fully functional OneDrive setup on Ubuntu, capable of syncing files from multiple accounts seamlessly.

## Tips and Tricks

### Syncing smarter, not harder

OneDrive can hold thousands of files, but not everything needs to be synced. When working with massive datasets or legacy documents, selectively syncing only the essentials can make a world of difference.

Use a `sync_list` file to define what should be synced:

```bash
# for your personal account
nano ~/.config/onedrive-personal/sync_list
```

This is how my `sync_list` looks:

```
# Exclude all .git directories
!.git/
# Exclude all renv directories
!renv/
# Other directories that I do not bother syncing
!/Public
!/Apps
!/'Email attachments'
!/Desktop                
# Specific directory with heavy files
!/2_areas/nematode-genomics/oncho_gis/
!ncbi_dataset/

# Important! Include desired directories

/1_projects
/2_areas
/3_resources
```

> ⚠️ Caution When Using `sync_list`

> The `sync_list` uses an **exclusion-first** approach. This means **only the items listed in `sync_list` will be synced**, and everything else is ignored.

> * Don't forget to include all folders you want to sync, or they won’t be downloaded/uploaded.
> * Here's an interesting discussion on the [GitHub issues page](https://github.com/abraunegg/onedrive/discussions/2369) that helped me understand this better.

> Always test your `sync_list` with a dry run first to avoid accidental data loss. So, you will have to resync with `--dry-run`:

```bash
onedrive --confdir="~/.config/onedrive-personal" --sync --resync --dry-run
```

---

## Using `--monitor` mode with caution

When you are manually syncing files, there are a few things to keep in mind while using the `--monitor` mode.

* It’s best **not** to leave `--monitor` running indefinitely.
* Once sync is done, turn it off.

```bash
onedrive --monitor --verbose --confdir="~/.config/onedrive-personal"
```

* Consider running sync/monitor in `--verbose` mode to see all the logs.

> **Important:** Monitor mode sometimes misses changes—especially deletions or files created online. In one case, files I deleted in the cloud were re-uploaded from my local machine because the deletion wasn’t properly detected.

---

## The Great `--resync` Caveat

If syncing gets messy, you might consider running a full `--resync`. But beware:

```bash
onedrive --resync --confdir="~/.config/onedrive-personal" --verbose
```

This can result in **thousands** of `.safebackup` files being created if conflicts are detected. These backup files will be uploaded to the cloud, potentially doubling or tripling your storage usage.

> **Tip:** Always back up your local OneDrive folder before using `--resync`. Watch the logs closely for what's happening.

---

## Troubleshooting Common Issues

### Problem: Constant `.safebackup` Files Being Created

* This often means `onedrive` detects mismatched timestamps or minor differences between local and cloud versions.
* Using `--disable-upload-validation` might reduce unnecessary safebackups, but be cautious, as it tells the client to trust the local version more aggressively.

### Problem: HTTP 503 Errors

* These are usually service-side throttling issues from Microsoft Graph API.
* Wait and retry; avoid running continuous syncs from multiple machines.

---

## Final Thoughts

Using OneDrive on Ubuntu isn’t as plug-and-play as on Windows, but with `onedrive` by abraunegg, it’s surprisingly powerful and customisable.

With careful configuration, a solid understanding of how the tool behaves, and some patience, you can sync your work and personal Microsoft 365 accounts on Ubuntu without a hitch.

It’s not perfect, and yes, you do have to get your hands a little dirty in the terminal, but the payoff is worth it. It’s fast, secure, and gives you complete control over your sync process.

---

Let me know in the comments if you have any tips or challenges of your own—we Linux users need to help each other out!

Happy syncing!
