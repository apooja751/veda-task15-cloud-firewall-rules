# Task 15: Configure Cloud Firewall Rules

**Cloud Computing Internship | Veda Technology**

## 1. Aim

To configure inbound and outbound firewall rules for a cloud virtual machine using AWS EC2 Security Groups and verify network access.

## 2. Objectives

- Understand basic network security in cloud environments.
- Configure inbound and outbound security group rules.
- Restrict SSH access to a trusted IP address.
- Allow HTTP traffic for a public web server.
- Test allowed and unconfigured ports.
- Understand why unnecessary ports should remain closed.

## 3. Tools and Technologies

- Amazon Web Services (AWS)
- Amazon EC2
- AWS Security Groups
- Ubuntu Server 26.04 LTS
- Nginx Web Server
- Windows PowerShell
- SSH

## 4. Environment Configuration

|      Component   |           Configuration              |
|------------------|--------------------------------------|
| Cloud Provider   | Amazon Web Services                  |
| Region           | Asia Pacific (Mumbai) — `ap-south-1` |
| EC2 Instance     | `veda-task15-firewall-vm`            |
| Operating System | Ubuntu Server 26.04 LTS              |
| Instance Type    | `t3.micro`                           |
| Security Group   | `veda-task15-firewall-sg`            |
| Root Volume      | 8 GiB gp3, encrypted                 |
| Web Server       | Nginx                                |
| Remote Access    | SSH                                  |

## 5. Firewall Rules

### Inbound Rules

Inbound rules control incoming connections to the EC2 instance.

| Type | Protocol | Port |         Source                    |          Purpose             |
|------|----------|------|-----------------------------------|------------------------------|
| SSH  | TCP      | 22   | Administrator's public IP (`/32`) | Secure remote administration |
| HTTP | TCP      | 80   | `0.0.0.0/0`                       | Public web access            |

SSH access is restricted to a specific IP address. HTTP is publicly accessible to allow users to access the demonstration website.

### Outbound Rules

|     Type    | Protocol | Destination |           Purpose                                                           |
|-------------|----------|-------------|-----------------------------------------------------------------------------|
| All traffic | All      | `0.0.0.0/0` | Allows outbound connections, such as package downloads and software updates |

The outbound rule is permissive. Production environments may require more restrictive outbound rules based on application requirements.

## 6. Implementation Procedure

1. Launched an Ubuntu EC2 instance in the Mumbai region.
2. Created the security group `veda-task15-firewall-sg`.
3. Configured SSH access on TCP port 22, restricted to the administrator's IP.
4. Reviewed the outbound security group rule.
5. Removed an overly permissive inbound rule that allowed all traffic from all IPv4 addresses.
6. Added an HTTP inbound rule on TCP port 80.
7. Connected to the Ubuntu instance from Windows PowerShell using SSH.
8. Verified the Linux username and hostname.
9. Installed Nginx and enabled its service.
10. Tested the public website using the EC2 instance's public IPv4 address.
11. Tested TCP port 8080 from Windows PowerShell.
12. Verified the final inbound firewall configuration.

## 7. Testing and Results

### Test 1: SSH Connection

Connected to the Ubuntu virtual machine using the private key file.

Commands:

```bash
whoami
hostname
```

Observed results:

- Username: `ubuntu`
- Hostname: `ip-172-31-10-0`

**Result:** SSH access succeeded from the permitted source IP.

### Test 2: Nginx Service

Commands:

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx --no-pager
```

**Result:** Nginx was enabled and its status showed `active (running)`.

### Test 3: Public HTTP Access

Opened the EC2 instance's public IPv4 address in a browser using HTTP.

**Result:** The Nginx welcome page loaded successfully, confirming that the website was reachable over TCP port 80.

### Test 4: Unconfigured Port 8080

Command executed in Windows PowerShell:

```powershell
Test-NetConnection <EC2-PUBLIC-IP> -Port 8080 -InformationLevel Detailed
```

**Observed result:** `TcpTestSucceeded` returned `False`.

This means the TCP connection attempt failed. The result is consistent with port 8080 not being available in this configuration. A failed connection alone does not prove that the security group was the only cause.

## 8. Understanding the Ports

- **Port 22 — SSH:** Used for remote administration of the Linux virtual machine. It should be restricted to trusted IP addresses.
- **Port 80 — HTTP:** Used to serve web pages. It is publicly accessible for this demonstration.
- **Port 8080 — Unconfigured:** No inbound rule or application service was configured for this port during the test.

Unnecessary open ports can expose services to unauthorized access and increase the attack surface.

## 9. Architecture Diagram

```text
Administrator Computer
        |
        | SSH - TCP 22
        | Restricted to administrator IP
        v
  AWS Security Group
        |
        v
    Ubuntu EC2 VM
        |
        +---- Nginx Web Server
        |          |
        |          +---- HTTP - TCP 80
        |                Public access
        |
        +---- TCP 8080
                   No inbound rule configured
```

## 10. Evidence Screenshots

The following screenshots document the configuration and testing performed in this practical.

|                        Screenshot                 |              Description                 |
|---------------------------------------------------|------------------------------------------|
| `Screenshorts/01_Inbound_Rules_Configuration.png` | Initial inbound SSH rule configuration   |
| `Screenshorts/02_Instance_Launch_Success.png`     | Successful EC2 instance launch           |
| `Screenshorts/03_Verified_Inbound_Rules.png`      | Verified inbound firewall rules          |
| `Screenshorts/04_Verified_Outbound_Rules.png`     | Outbound security group rule             |
| `Screenshorts/05_SSH_Connection.png`              | Successful SSH login                     |
| `Screenshorts/06_SSH_Connection_Verification.png` | Linux username and hostname verification |
| `Screenshorts/07_HTTP_Inbound_Rule.png`           | HTTP inbound rule configuration          |
| `Screenshorts/08_Nginx_Service_Running.png`       | Nginx service status                     |
| `Screenshorts/09_Public_HTTP_Test.png`            | Public Nginx welcome page                |
| `Screenshorts/10_Blocked_Port_Test.png`           | Failed TCP port 8080 connection attempt  |
| `Screenshorts/11_Final_Firewall_Rules.png`        | Final inbound firewall configuration     |

The screenshots are stored in the Screenshorts/ directory.

## 11. Security Best Practices

- Allow only required inbound ports.
- Restrict SSH to trusted IP addresses.
- Avoid allowing all inbound traffic from `0.0.0.0/0`.
- Use encrypted storage where possible.
- Verify security group rules after modifications.
- Keep private keys and AWS credentials secret.
- Review outbound permissions according to application requirements.
- Remove temporary resources after completing the task.

## 12. Rollback and Cleanup

### Rollback procedure

To remove public web access after testing:

1. Open **EC2 → Security Groups** in the AWS Console.
2. Select `veda-task15-firewall-sg`.
3. Choose **Edit inbound rules**.
4. Remove the HTTP rule for TCP port 80.
5. Save the changes.
6. Verify that only the required SSH rule remains.

This procedure describes the rollback steps. A separate rollback demonstration should be performed and captured if the evaluator requires rollback evidence.

### AWS cleanup

After saving all evidence and completing the documentation:

1. Terminate the temporary EC2 instance.
2. Verify that its root EBS volume is deleted as configured.
3. Check for unused volumes, snapshots, Elastic IPs, or other billable resources created for this task.
4. Confirm that no unnecessary resources remain.

## 13. Conclusion

Successfully configured and verified AWS EC2 Security Group rules for SSH and HTTP access. SSH was restricted to a trusted IP address, HTTP was enabled for the Nginx web server, and a connection attempt to the unconfigured port 8080 failed. This practical demonstrated the importance of allowing required network traffic while limiting unnecessary access.

