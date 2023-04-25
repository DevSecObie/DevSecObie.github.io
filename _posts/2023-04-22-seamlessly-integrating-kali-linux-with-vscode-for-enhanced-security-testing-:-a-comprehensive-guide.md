---
title: Seamlessly Integrating Kali Linux with VSCode for Enhanced Security Testing 
date: 2023-04-22 11:00:28 -0700
categories: [Productivity, development, pentesting]
tags: [vscode, tricks, virtual machine]
---

This guide aims to provide a step-by-step tutorial on seamlessly integrating [Kali Linux](https://www.kali.org/) with [Visual Studio Code (VSCode)](https://code.visualstudio.com/) to enhance security testing capabilities for cybersecurity professionals, ethical hackers, and developers interested in security testing. By following this guide, the intended audience will be able to optimize their workflow by accessing Kali Linux tools directly within VSCode, reducing the need to switch between multiple environments while performing various security tests.

![Kali Linux & VSCode](https://i.imgur.com/irFVouK.png)

## Prerequisites

1. **Knowledge:**
    - Basic understanding of Linux command-line operations.
    - Basic Understanding of Networking and Security
    - Familiarity with the Visual Studio Code interface and it's extensions system.
    - Some experience with security testing or ethical hacking concepts is helpful but not required.

2. **System Requirements:**
    - A compueter with a stable internet connection to download and install required software and extensions.
    - A modern operating system (windows 10/11, macOS, or linux) that supports Kali Linux, Virtual Studio Code, and Virtualization.
    - Suffiecient system resources (CPU,RAM, and Storage) to run Kali Linux tools and Visual Studio Code effeciently.

3. **Virtual Machine:**
    - You can use a server in the cloud to achieve this as well, there are many cloud providers that provide linux machines.
    - This guide has been tested using [Parallels Desktop](https://www.parallels.com/products/desktop/) and [Virtual Box](https://www.virtualbox.org/wiki/Downloads) as the VM providers running Kali Linux. While other virtualization software like [VMware](https://customerconnect.vmware.com/en/downloads/info/slug/desktop_end_user_computing/vmware_workstation_pro/16_0) may work, the specific setup and configuration steps may differ slightly. 
    
    > I have NOT tested this on `Kali Linux subsystem for Windows (WSL)` if you are using Windows 10 or later.
    {: .prompt-warning}

    > If you do find a way, that would be nice to share 😃
    {: .prompt-tip}

4. **Software:**
    - [Prebuilt Virtual Machines](https://www.kali.org/get-kali/#kali-virtual-machines) would be great for a quick start and you could scale your resources to fit your needs after set up.

    - If you already have a Linux virtual machine you can skip to where you see fit.
    
    - Install Visual Studio onto your Local machine.


## Step 1: Installing Kali Linux and VSCode

> In This Demonstraion we will be using the prebuilt Linux virtual machine
{: .prompt-info}

1. Select VirtualBox (link to download provided above). [docs](https://www.kali.org/docs/virtualization/install-virtualbox-guest-vm/)

    ![Kali Linux Prebuilds](https://i.imgur.com/tT4ivzg.png)_Select your Linux virtual machine_

2. Download and install VirtualBox (link to download provided above), and select the Platform package according to your system.
   
    ![VirtualBox](https://i.imgur.com/V1q6ngV.png)

3. Download and install Visual Studio code 

    ![Visual Studio Code](https://i.imgur.com/dH3lvfy.png)_Download is automatically configured to your system._

4. After All Files have been downloaded, unzip your VirtualBox VM from Kali using a unzip utility tool. Mac has Archive utility that automatically unzips, Windows has 7zip, winrar and many others in their store.

    ![Unzip](https://i.imgur.com/dv3LYV6.png)

    - A folder containing the VM should appear next to the zip file. Go into that folder.
        
        ![Folder](https://i.imgur.com/LUcfezS.png)

    - Inside that folder you should see two files, one ending with .vbox and the other .vdi, select .vbox and Virtual Box should open with the preset VM ready to start.

        ![Files](https://i.imgur.com/ED5Pr7o.png)


> Disregard my `"can't open machine"` error, I already have it installed and it can't open a file with the same name/UUID.
{: .prompt-info}


- Click settings, and navigate to `Network`, check the box `[✔] Enable Network Adapter` in the `Attached to:` dropdown list select `Bridged Adapter` in the `Name` menu select `Wi-Fi`.


    ![Network Settings](https://i.imgur.com/156ipMj.png)


Here's a brief explanation of the network settings we configured:

1. **Enable Network Adapter:** This setting allows you to enable or disable the network adapter for your virtual machine. Enabling the network adapter ensures that your virtual machine can communicate with the host machine and external networks.

2. **Attached to:** This option determines the type of network connection your virtual machine will have. The available options are NAT, Bridged Adapter, Internal Network, and Host-only Adapter.

3. **Bridged Adapter:** By selecting the Bridged Adapter option, you connect your virtual machine to the same network as your host machine, giving it access to the same resources and network devices. This configuration allows the virtual machine to appear as a separate device on the network, with its own IP address.

4. **Name:** The name field specifies the network interface on the host machine that the virtual machine will use. In this case, you have selected en0: Wi-Fi, which is the Wi-Fi interface on the host machine.

5. **Advanced:** The Advanced settings provide additional options for configuring the network adapter, such as MAC address, adapter type, and other advanced features. These settings allow for further customization and optimization of the network connection for your specific use case.

We've configured the virtual machine to use a bridged network adapter, connecting it to the same network as the host machine through the Wi-Fi interface. This configuration allows the virtual machine to communicate with the host machine and other devices on the network as if it were a separate physical device.

- Now go back to the `Oracle VM VirtualBox Manager` double click on your Virtual Machine to boot. Close the Manager and wait for the VM to boot up.

    ![VM VirtualBox Manager](https://i.imgur.com/kxBhcNy.png)


## Step 2: Configuring Kali Linux in VSCode Terminal
- Login to the Virtual Machine using the default credentials. Username `kali` and password `kali`.
    
    ![Login](https://i.imgur.com/F0ByZam.png)

- In the Terminal, we are going to **Create a user other than root**
    - `sudo adduser username`
        - ![adduser](https://i.imgur.com/hw8dFqx.png)
    - `sudo usermod -aG sudo username`
        - ![adduser](https://i.imgur.com/NHzjL49.png)
    - `sudo su`
        - su means to switch users (for those that are new)

- In the Terminal, type `ip a` and take note of the IP address in the `eth0` network interface.

    ![ip a](https://i.imgur.com/DrymumY.png)

- Next we want to configure the system to allow incoming SSH connections, we are going to configure the `/etc/ssh/sshd_config` file. This file contains the settings for the SSH daemon (sshd) responsible for handling incoming SSH connections.
    - check ssh status with `sudo systemctl status ssh`
    - if you don't have it on your system, use 
        - `sudo apt update` and 
        - `sudo apt install openssh-server` to install.
    - After it is installed, navigate to `/etc/ssh/sshd_config`

    ![sshd_config](https://i.imgur.com/rrBHUh5.png)

    - Next we are going to enable SSH connections by removing the `#` from in front of the port number. This will allow incoming SSH connections on port 22. But we are going to use a different port. You can use port 22 if you want but to ensure more secure connections I'd recommend changing it.
    
    > Please note that you should be cautious when making changes to the SSH configuration, as incorrect settings may lock you out of the system. Always double-check your changes and ensure you have a backup access method (such as creating another user or console access if your're using the cloud).
    {: .prompt-danger}

    ![Port](https://i.imgur.com/0QTFpbJ.png)

    - When changing the listening port for the SSH server in the `/etc/ssh/sshd_config` file, you can choose a port number within the valid range of 1 to 65535. However, it is recommended to avoid well-known ports (1-1023) and registered ports (1024-49151) that are reserved for specific services or applications.

        Typically, a port number in the range of 49152 to 65535, known as the dynamic or private port range, is a good choice for custom SSH configurations. These ports are less likely to conflict with other services or be targeted by automated attacks.

        For example, you can change the SSH listening port to 3500.
    
    ![Imgur](https://i.imgur.com/ayYpOk7.png)

    > While port 3500 is not in the dynamic or private port range (49152-65535), it is less likely to be targeted by automated attacks compared to the default SSH port (22). Always double-check if the port that you choose is not being utilized by other services or applications on your system to avoid conflicts.
    {:.prompt-info}

    - Also in the config file, change these settings

        - `PermitRootLogin no`
        - `PasswordAuthentication yes`
        - `PermitEmptyPasswords no`
        - `PubkeyAuthentication yes`

> When connecting to a virtual machine (VM) on your local computer, it's still advisable to use SSH keys for secure access, regardless of whether you're using a terminal or an application like VSCode. However, keep in mind the potential risk of a compromised local machine. If an unauthorized user obtains your private key, they could access your VM. To reduce this risk, protect your private key with a strong passphrase. Additionally, consider implementing two-factor authentication (2FA) as an extra security measure, although enabling 2FA for SSH connections may require extra configuration or tools. In summary, using SSH keys, a robust passphrase, and considering 2FA are essential practices for securely accessing your local VM.
{: .prompt-info}


- Exit the text editor and restart ssh.

    - `sudo systemctl restart ssh`
    
    
    - Check SSH status 

        - `sudo systemctl status ssh`

            ![ssh status](https://i.imgur.com/Wti3KC9.png)

- Now we will install a firewall, we will use `ufw`. (Uncomplicated Firewall) is a user-friendly front-end for managing iptables firewall rules on Linux systems.

    - `sudo apt update && sudo apt install ufw`

        ![ufw](https://i.imgur.com/reAVDjf.png)

    - Check the status of the firewall
        - `sudo ufw status`
        It might be off.
        - `sudo ufw enable`
        - `sudo ufw allow <port><protocol>`
            - Allow incoming traffic on a specific port and protocol (e.g., `sudo ufw allow 3500/tcp).` 
             **add the port you changed here**
    - Check the status Again
        - `sudo ufw status`

        ![ufw status](https://i.imgur.com/JwWUUhw.png)


**Great! now minimize your VM and open VSCode**


## Step 3: Installing and Configuring Essential Plugins

- Now we will install Remote-SSH from the extension manager.

    ![Remote-SSH](https://i.imgur.com/rUFtYYj.png)

- There are MANY plugins and extensions available that I may go over in the future, but for now install remote-ssh.
    
    ![Remote-plugins](https://i.imgur.com/ZlwgSRd.png)

- After it's installed, click on the symbol underneath the settings gear.
    
    ![symbol](https://i.imgur.com/vqRMWI1.png)

- Click Connect to Host... or Connect Current Window to Host... either option works.
    
    ![connect](https://i.imgur.com/APkBaoF.png)

- As you can see, `Kali` and other Hosts there. You could use VSCode to manage your servers, VMs, Docker container etc. for now we will click `+ Add New SSH Host`

    ![New SSH Host](https://i.imgur.com/1acXGUR.png)

- A new window will appear, add this into the SSH Connection Command bar: `ssh -p <port> username@IP_Address`

    ![ssh command](https://i.imgur.com/pb0vmjg.png)

    
    **It's looking good!!!**

   ![Host added](https://i.imgur.com/mSdLZx4.png)

   
   **SUCCESS!!!! You did it!**

- Now, on the left side bar is the `Remote Explorer`, looks like a little TV screen 📺. Click that, and as you can see the settings gear, select that and a window will appear to configure your ssh file.

    ![Remote Explorer](https://i.imgur.com/vmE9nLO.png)

- Or you can click the bottom left corner of the window

    ![left corner](https://i.imgur.com/CaYyXUn.png)

- Select `Configure SSH Hosts`

    ![SSH Config](https://i.imgur.com/jUxxLma.png)

- Change your "Host" name to whatever you like.
    
    ![hostname](https://i.imgur.com/9WbVw34.png)


**NOW LET THE PARTY BEGIN!!!**

## Step 4: Setting up a Secure Environment

1. 2-Factor Authentication: Two-factor authentication (2FA) adds an extra layer of security to your accounts by requiring a second form of verification, usually a code generated by an app on your mobile device or sent via SMS. To enable 2FA for your online accounts, visit the security settings of the service you want to secure (e.g., Google, GitHub, etc.), and follow their instructions for setting up 2FA. For SSH, you can use Google Authenticator or a similar app to enable 2FA. Install the required packages (libpam-google-authenticator and libqrencode3), run the google-authenticator command, and follow the prompts. Then, update your /etc/pam.d/sshd and /etc/ssh/sshd_config files to enable 2FA for SSH connections.

Here's a step-by-step guide on how to install and configure Google Authenticator for SSH using the PAM module:

Update your package list:
```bash
sudo apt update
```
Install the libpam-google-authenticator package:

```bash
sudo apt install libpam-google-authenticator
```
Run Google Authenticator as your user (not root):
```bash
google-authenticator
```
Follow the prompts, and make sure to save the generated emergency scratch codes.

Scan the generated QR code with your Google Authenticator app on your phone or enter the provided secret key manually.

Edit the PAM configuration for SSH:

```bash
sudo nano /etc/pam.d/sshd
```
Add the following line to the top of the file:

```plaintext
auth required pam_google_authenticator.so
```
Press `Ctrl + X` to exit, then `press Y` and `Enter` to save the changes.

Edit the SSH daemon configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Find the line that says ChallengeResponseAuthentication and change it to:

```plaintext
ChallengeResponseAuthentication yes
```

If the line doesn't exist, add it to the file.

Save and exit the file by pressing `Ctrl + X`, then `press Y` and `Enter`.

Restart the SSH service:

```bash
sudo systemctl restart ssh
```

Now, when you log in via SSH, you'll be prompted for your password and a verification code generated by the Google Authenticator app on your phone.

---



2. Hardening process: Hardening is the process of securing a system by reducing its attack surface and minimizing potential vulnerabilities. Here are some steps you can take to harden your system:
    - Keep your system updated: Regularly apply security patches and updates to your operating system and installed software.
    - Configure a firewall: Use a firewall like ufw (Uncomplicated Firewall) to restrict incoming and outgoing network traffic based on rules you define.

    - Limit user access: Grant the minimum necessary privileges to users and applications, and avoid using the root account for everyday tasks.
Disable unnecessary services: Disable any services and applications that are not required for your system's intended purpose.
    - Monitor system logs: Regularly review system logs to identify potential security incidents or unauthorized access attempts.
    - Use strong, unique passwords: Use a password manager to generate and store strong, unique passwords for each of your accounts.
    - Implement intrusion detection and prevention systems (IDS/IPS): Tools like Snort, Suricata, or Fail2Ban can help detect and prevent unauthorized access or malicious activity on your system.


    `sudo apt install auditd`
    Auditd is the userspace component of the Linux Auditing System. It's a system service that captures and logs information about events and activities happening on a Linux system. This information is valuable for security and compliance purposes, as well as for troubleshooting and monitoring user activities.

## Step 5: Performing Security Testing within VSCode

By integrating Kali Linux with Visual Studio Code, you can navigate through the system more efficiently than you would by doing it manually. This streamlined workflow enables you to quickly identify and address issues, such as traceback errors. When errors occur, you can directly open them in a separate window within VSCode, making it easier to debug and resolve problems without leaving your environment. This enhances productivity and saves valuable time compared to traditional manual approaches.

![Autopsy](https://i.imgur.com/8i7EfQ6.png)


You can easily transfer files directly from your local machine to your virtual machine. This capability simplifies the process of working with files across different environments and ensures a smooth workflow when using tools like Kali Linux and Visual Studio Code.


![Copy & Paste](https://i.imgur.com/7XfNJWF.png)


With Metasploit, you can open multiple shells simultaneously, providing you with the flexibility to manage various tasks and sessions in parallel. While there may not be a hard limit on the number of shells you can open, it's essential to consider your system's resources and performance to ensure an efficient and stable experience.


![Mfsconsole](https://i.imgur.com/Ge523o0.png)


I created this blog using the technique of integrating Kali Linux with Visual Studio Code, and I was able to stay within VSCode throughout the entire process, thanks to some of their helpful plugins. This seamless integration allowed me to work efficiently and productively without the need to switch between multiple tools or platforms.

![Blog](https://i.imgur.com/axcgATw.png)


**There's a whole world of amazing features and possibilities waiting for you when you integrate Kali Linux with Visual Studio Code. This powerful duo caters to developers, security experts, and tech enthusiasts, offering an unparalleled experience that boosts productivity and takes your skills to the next level. So, what are you waiting for? Dive in and explore this fantastic combination! You'll be amazed at how much easier and enjoyable your cybersecurity journey becomes. Embrace the incredible synergy between Kali Linux and Visual Studio Code and see the difference for yourself!**

## Step 6: Troubleshooting and Common Issues
- 
I have not encountered any issues so far. However, if you face any problems, try terminating processes or shutting down your virtual machine if you have opened too many windows while remotely logging into your account.

> Your Virtual Machine must be running in order to access it via SSH. Essentially think of it as hosting your own server, if it is shut down you can't access it. I'd recommend using a cloud provider if you don't have a lot of resources (CPU, RAM, Storage) on your local machine.
{: .prompt-tip}

## Conclusion
- In conclusion, integrating Kali Linux with Visual Studio Code (VSCode) offers numerous benefits to developers, security professionals, and technology enthusiasts alike. This powerful combination streamlines your workflow, enhances productivity, and facilitates a more efficient use of time and resources.

Some key benefits include:

- Centralized Workspace: Combining Kali Linux's robust security toolset with VSCode's intuitive interface creates a unified workspace, allowing users to manage tasks efficiently without switching between multiple tools or platforms.
- Customization: Both Kali Linux and VSCode offer extensive customization options, enabling users to tailor their environment to their specific needs and preferences.
- Language Support: With VSCode's extensive support for various programming languages, users can easily work on multiple projects in different languages within a single environment.
- Extensions and Plugins: The availability of numerous extensions and plugins for both platforms enhances their capabilities, making it easier to integrate essential tools and features into the development process.
- Community Support: Both Kali Linux and VSCode boast strong communities, ensuring you have access to a wealth of knowledge, resources, and assistance when needed.

By integrating Kali Linux with VSCode, you can leverage the best of both worlds, creating a seamless, efficient, and powerful development experience. Give it a try and witness the incredible synergy between these two remarkable tools.

## Additional Resources
[hackr.io - Best VSCode Extensions](https://hackr.io/blog/best-vscode-extensions)

[Install Virtualbox guest VM](https://www.kali.org/docs/virtualization/install-virtualbox-guest-vm/)

[Kali Linux Windows Subsystem Linux (WSL)](https://apps.microsoft.com/store/detail/kali-linux/9PKR34TNCV07?hl=en-us&gl=us&SilentAuth=1&wa=wsignin1.0)

[Best Security Apps for VSCode by Yoda](https://devdojo.com/yoda/top-vs-code-extensions-for-application-security-in-2021)

[VSCode Tips & Tricks](https://code.visualstudio.com/docs/getstarted/tips-and-tricks)


