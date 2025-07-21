# Jenkins_on_EC2
**Title:** Hosting Jenkins on an AWS EC2 Instance (with Troubleshooting)

**Author:** Sukumar Chinthalapudi
**Date:** July 21, 2025

---

## Objective

To deploy Jenkins on an Ubuntu-based AWS EC2 instance, set it up for CI/CD, and resolve installation issues step by step.

---

## Step 1: Launch EC2 Instance

1. **AMI:** Ubuntu Server 22.04 LTS (64-bit)

2. **Instance Type:** t2.medium or higher (minimum 2 GB RAM)

3. **Security Group Settings:**

   * SSH (Port 22) from My IP
   * HTTP (Port 80) from 0.0.0.0/0
   * Custom TCP (Port 8080) from 0.0.0.0/0 (for Jenkins)

4. **SSH into the instance:**

```bash
ssh -i your-key.pem ubuntu@<public-ip>
sudo -i  # switch to root
```

---

## Step 2: Install Java (Required for Jenkins)

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version  # confirm installation
```

---

## Step 3: Install Jenkins (APT-based - FAILED path)

```bash
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
```

### ❌ Issue #1: Jenkins failed to start

```bash
systemctl status jenkins
# Output showed: Failed with result 'exit-code', restart count exceeded
```

### 🛠️ Fix Attempt 1:

* Checked `/var/log/jenkins/jenkins.log` -> File did not exist
* `/usr/bin/jenkins` existed but was not the correct binary
* Service file pointed to incorrect executable

---

## Step 4: Manual Jenkins Installation (WORKING)

### ✅ Create directory and download Jenkins WAR:

```bash
sudo mkdir -p /opt/jenkins
cd /opt/jenkins
wget https://get.jenkins.io/war-stable/2.440.2/jenkins.war
```

### ✅ Run Jenkins manually:

```bash
java -jar jenkins.war
```

* Jenkins successfully started on port `8080`
* Unlocked using admin password from:

```bash
cat ~/.jenkins/secrets/initialAdminPassword
```

---

## Step 5: Access Jenkins Web UI

```url
http://<your-ec2-public-ip>:8080
```

* Used the above URL to access Jenkins setup wizard
* Selected **"Install suggested plugins"**
* Plugins installed successfully

---

## Step 6: (Optional) Create Jenkins systemd service

```bash
sudo nano /etc/systemd/system/custom-jenkins.service
```

Paste:

```ini
[Unit]
Description=Custom Jenkins Server
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/bin/java -jar /opt/jenkins/jenkins.war
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable custom-jenkins
sudo systemctl start custom-jenkins
```

---

## Summary of Issues & Fixes

| Issue                              | Description                     | Fix                          |
| ---------------------------------- | ------------------------------- | ---------------------------- |
| Jenkins failed to start            | `systemctl` showed exit-code    | WAR file was missing         |
| Log file missing                   | No output at `/var/log/jenkins` | Jenkins wasn't even starting |
| Wrong binary in `/usr/bin/jenkins` | Wasn't the actual app           | Used `jenkins.war` directly  |

---

## Final Result

* Jenkins successfully hosted on AWS EC2
* Web interface working at `http://3.83.249.153:8080`
* Admin password retrieved
* Plugins installed and ready for job setup

---

## Author Notes

This doc captures the real-world debugging process during Jenkins deployment. Useful for anyone trying to self-host Jenkins on a fresh cloud VM.
