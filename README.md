<h1>Configuring Basic Security Controls on a CentOS Linux Server</h1>


<h2>Description</h2>
To some extent, information security on Linux systems is no different than information security on Microsoft or Apple systems. You’ll take many of the same steps that you would in every other system. You’ll create firewalls, set up bastions, manage users, encrypt file systems, configure servers, customize applications, and work with intrusion prevention and detection systems. The Linux paradigm is different in some ways, however. With Linux, you can customize the kernel and set up mandatory access with Application Armor (AppArmor) or Security Enhanced Linux (SELinux), a set of security enhancements built into the Linux kernel.

 

In this lab, I secured a Linux server system. I secured the bootloader, enabled iptables firewall, and ran SELinux to help lock down the Linux OS. By securing the bootloader I was able to prevent access to single user mode and the GRUB (Grand Unified Bootloader) Console during the boot of the system. Enabling iptables and applying firewall rules ensured that only the applications I trust have the ability to reach into or out from your computer. I also applied access control lists (ACLs) to directories and files within the lab to secure the file and data access and then verified those permissions on the system.
<br />


<h2>Network Topology</h2>

- TargetLinux01 (CentOS Linux)
- TargetLinux02 (Xubuntu Linux)</b>
- ![image](https://github.com/user-attachments/assets/f1694381-5810-4ad1-a768-af4577164aba)

<h2>Tools and Software</h2>

- iptables
- sysv-rc-conf
- Terminal
- vi Editor

<h2>Lab walk-through:</h2>

In the following steps, I hardened security measures on this server by modifying GRUB, the Grand Unified Bootloader, so that it will boot into the default operating system without displaying the boot loader menu at start-up. A boot loader is a program that allows the user or administrator to choose which operating system or kernel to load when the computer starts. The boot loader menu displays the available operating systems. GRUB can load a variety of free operating systems, as well as proprietary operating systems with chain-loading. It is important to lock down GRUB with a short delay and password credential to avoid tampering with the operating system’s boot sequence. Securing the bootloader also provides a layer of security from those who may have physical access to the machine.

Generated an encrypted password for grub2 bootloader and updated the configuration file with the encrypted password:
 
  ![image](https://github.com/user-attachments/assets/2b019f8b-b92f-4e6e-a877-79138271e1e8)
  ![image](https://github.com/user-attachments/assets/dbf42bce-bcd3-4ba9-b1ea-5c4523dd5f86)
  
<br />
Removed the option for all users to access the boot loader menu:

![image](https://github.com/user-attachments/assets/175c4f41-3369-44b4-ab67-61d8c4748a08)

Although I'm able to view the boot loader menu, any attempt to select an OS, edit the boot options, or open a command prompt will require the Grub2 credentials that I set earlier in this lab.

![image](https://github.com/user-attachments/assets/eb94f299-a372-4d7f-8b59-5bec7ccb6f2c)

In the next step, I hardened security measures on this server by verifying that this CentOS SELinux (Security-Enhanced Linux) server is set to enforcing mode. SELinux is a set of security enhancements built in to the Linux kernel and should be set to enforcing mode by default.

Used the command cat /etc/selinux/config to check the SELinux configuration and the /usr/sbin/sestatus to see which mode SELinux is running

![image](https://github.com/user-attachments/assets/842e4e08-5211-4b9b-b74c-163d57237197)

Next, I hardened security measures on this server by enabling the internal host-based IP stateful firewall, iptables. I set the firewall to start when it enters runlevel 2. iptables controls incoming and outgoing network connections and either allows, denies, or forwards requests based on a set of defined rules that you configure within the firewall application itself.

Used yum install iptables-services to install the firewall, /sbin/service iptables start to start the iptables firewall, systemctl enable iptables to enable iptables, and systemctl status iptables to view the current iptables configuration. At the command prompt used the iptables -L command to list the current iptables policy list.

![image](https://github.com/user-attachments/assets/2d868d27-fd81-4ad4-bbff-db53b220e29c)

Futhermore, I hardened security measures on this server by enabling the sudo user account. Administrators may choose to grant authorized users limited root level access for a specific purpose. Sudo access also can be used to allow the administrator to audit the actions that authorized users take on the server. Any actions not specifically granted to the sudo user will require the actual root password. Sudo access allows a user to perform actions beyond his or her normal privileges assigned by the ACLs. It is a way to provide these privileges to a user without requiring him to log in as root. A compromised root user account will allow an attacker to compromise an entire Linux server and system. By not logging in as root user, your IP packets cannot be compromised, and it’s not possible for a hacker to conduct a Man-in-the-Middle attack.

Used visudo to edit the sudoers file, the configuration file for the sudo users. First, I activated several command aliases, then I located and copied the following line:

 

# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

 

I pasted the line twice in the file, changing the %sys designation to %wheel and %web to assign sudo access for each of the commands listed here to the wheel and web groups. Wheel is a pre-configured group that has access to all commands by default. To improve security controls on the CentOS Linux server, I explicitly defined the commands for the wheel group, then deactivated the default command access. Web is a new group.

New command assignments:

![image](https://github.com/user-attachments/assets/57bedbee-5a42-424c-aa2b-3bfef94fd69c)


Next, I hardened security measures on this server by setting the immutable permission on a temporary file. An immutable designation means the file cannot be manipulated. The file becomes a read-only file that cannot be deleted, moved, or edited; even by someone with root level access. The immutable permission is an extended file attribute, beyond the usual read/write sort of access. Other extended file attributes include: append only (a), secure deletion (s).

Created a temp file with touch /tmp/mytest and changed the file attributes with chattr +i /tmp/mytest and added the immutable flag (i) to the file permissions. Used lsattr /tmp/mytest to list the file attributes and verify that the immutable flag has been added to the file permissions.

![image](https://github.com/user-attachments/assets/771262e1-8528-4b9a-b93b-e85de16aab69)

Finally, I hardened security measures on this server by granting special permissions to allow a user access to the server’s log file. I assigned access control list permissions (ACL) to the wheel group. This was be done using the set file access control lists (setfacl) command and the get file access control lists (getfacl) commands. The setfacl command provides an extra layer of security defense by setting permissions for specific users or groups of users regarding access to files on the system. Setfacl helps system administrators define stringent access controls to specific files for ensuring confidentiality. The get file access control lists (getfacl) command displays existing ACLs.

Used the command getfacl /var/log/messages to view the current ACL permissions for the log file. Added read ACL permissions to the wheel group for the log file with setfacl -m g:wheel:r /var/log/messages.Got the new ACL permissions for the log file and verified that they were set properly with getfacl /var/log/messages.

Updated permissions for the log file:

![image](https://github.com/user-attachments/assets/c6bd3132-fa16-4bea-8395-0dea799e2465)

<br />

