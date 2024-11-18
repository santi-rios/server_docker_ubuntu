sudo apt-get install uidmap

Configuration of subordinate UIDs/GIDs: Ensure that /etc/subuid and /etc/subgid contain a sufficient number of subordinate UIDs/GIDs for the user who will be running Docker in Rootless mode

1. Check Current User’s UID/GID:

First, determine the UID and GID of the user who will be running Docker in Rootless mode. You can do this using the id command:

id -u <username>
id -g <username>


With Packages (RPM/DEB)

If you’re using Docker version 20.10 or later and have installed it using RPM or DEB packages, setting up Docker in Rootless mode is straightforward:

Run the below command as a non-root user to set up the daemon.

 dockerd-rootless-setuptool.sh install

Table of Contents[](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#)

-   [Understanding Rootless Mode](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Understanding_Rootless_Mode "Understanding Rootless Mode")
-   [Prerequisites](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Prerequisites "Prerequisites")
    -   [Debian/Ubuntu](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#DebianUbuntu "Debian/Ubuntu")
    -   [Red Hat/CentOS](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Red_HatCentOS "Red Hat/CentOS")
    -   [Fedora](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Fedora "Fedora")
    -   [Arch Linux](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Arch_Linux "Arch Linux")
-   [1\. Check Current User’s UID/GID](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#1_Check_Current_Users_UIDGID "1. Check Current User’s UID/GID")
-   [2\. Allocate Subordinate UIDs/GIDs](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#2_Allocate_Subordinate_UIDsGIDs "2. Allocate Subordinate UIDs/GIDs")
-   [3\. Add Entries for the User](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#3_Add_Entries_for_the_User "3. Add Entries for the User")
-   [4\. Save and Exit](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#4_Save_and_Exit "4. Save and Exit")
-   [5\. Verify](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#5_Verify "5. Verify")
    -   [Install the dbus-user-session Package](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Install_the_dbus-user-session_Package "Install the dbus-user-session Package")
-   [Installation Steps](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Installation_Steps "Installation Steps")
    -   [With Packages (RPM/DEB)](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#With_Packages_RPMDEB "With Packages (RPM/DEB)")
    -   [Without Packages](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Without_Packages "Without Packages")
    -   [Install the docker-ce-rootless-extras Package Manually](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Install_the_docker-ce-rootless-extras_Package_Manually "Install the docker-ce-rootless-extras Package Manually")
    -   [Set Up Environment Variables (PATH and DOCKER\_HOST) Manually](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Set_Up_Environment_Variables_PATH_and_DOCKER_HOST_Manually "Set Up Environment Variables (PATH and DOCKER_HOST) Manually")
-   [Usage and Best Practices](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Usage_and_Best_Practices "Usage and Best Practices")
-   [Troubleshooting](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Troubleshooting "Troubleshooting")
-   [Conclusion](https://collabnix.com/how-to-run-docker-in-a-rootless-mode/#Conclusion "Conclusion")

Whenever you install Docker on the Ubuntu or Debian system using the following command:

```
 curl -sSL https://get.docker.com/ | sh
```

You might encounter the following message at the end of the command that says “To install Docker in a non-privileged user, consider setting up Docker daemon in rootless mode for your user”. What does it mean?

[![docker installation](https://i0.wp.com/media.dev.to/cdn-cgi/image/width%3D800%252Cheight%3D%252Cfit%3Dscale-down%252Cgravity%3Dauto%252Cformat%3Dauto/https%253A%252F%252Fdev-to-uploads.s3.amazonaws.com%252Fuploads%252Farticles%252Fyn5kmyg0zazgqwaqooki.png?ssl=1)](https://i0.wp.com/media.dev.to/cdn-cgi/image/width%3D800%252Cheight%3D%252Cfit%3Dscale-down%252Cgravity%3Dauto%252Cformat%3Dauto/https%253A%252F%252Fdev-to-uploads.s3.amazonaws.com%252Fuploads%252Farticles%252Fyn5kmyg0zazgqwaqooki.png?ssl=1)

By default, Docker requires root privileges to run, as it interacts with system-level components such as the Docker daemon and system resources like networking and storage. However, running Docker as root can pose security risks, as any compromise of the Docker daemon could potentially lead to broader system compromise.

To mitigate these risks and allow non-privileged users to run Docker without granting them root access, Docker introduced a feature called “rootless mode.” In [rootless mode](https://docs.docker.com/engine/security/rootless/#prerequisites), the Docker daemon and containers run entirely within the user’s own namespace, without needing root privileges. In this guide, we’ll explore how Rootless mode works, its prerequisites, installation steps, known limitations, and best practices.

## [](https://dev.to/new#understanding-rootless-mode)Understanding Rootless Mode

Rootless mode operates by executing the Docker daemon and containers within a user namespace. This means that both the daemon and the containers run without root privileges, thus reducing potential security vulnerabilities. Unlike userns-remap mode, where the daemon runs with root privileges, Rootless mode ensures that all components operate within the confines of user permissions.

## [](https://dev.to/new#prerequisites)Prerequisites

Before setting up Docker in Rootless mode, there are several prerequisites to consider:

-   Installation of newuidmap and newgidmap: These commands, typically provided by the uidmap package, are necessary to manage user and group ID mappings within the user namespace.

### [](https://dev.to/new#debianubuntu)Debian/Ubuntu:

```
sudo apt-get install uidmap
```

### [](https://dev.to/new#red-hatcentos)Red Hat/CentOS:

```
sudo yum install uidmap
```

### [](https://dev.to/new#fedora)Fedora:

```
sudo dnf install uidmap
```

### [](https://dev.to/new#arch-linux)Arch Linux:

```
sudo pacman -S uidmap
```

Choose the appropriate command based on your distribution, and it will install the uidmap package, thereby providing the newuidmap and newgidmap commands required for Docker’s Rootless mode.

-   Configuration of subordinate UIDs/GIDs: Ensure that `/etc/subuid` and `/etc/subgid` contain a sufficient number of subordinate UIDs/GIDs for the user who will be running Docker in Rootless mode.

To configure subordinate UIDs/GIDs for the user who will be running Docker in Rootless mode, you need to ensure that the `/etc/subuid` and `/etc/subgid` files contain a sufficient number of subordinate UIDs/GIDs allocated for that user. Here’s how you can do it:

## [](https://dev.to/new#1-check-current-users-uidgid)1\. Check Current User’s UID/GID:

First, determine the UID and GID of the user who will be running Docker in Rootless mode. You can do this using the id command:

```
id -u &lt;username&gt;
id -g &lt;username&gt;
```

## [](https://dev.to/new#2-allocate-subordinate-uidsgids)2\. Allocate Subordinate UIDs/GIDs:

Edit the `/etc/subuid` and `/etc/subgid` files to allocate a range of subordinate UIDs/GIDs for the user. You’ll typically use a text editor like `nano` or `vim` to modify these files:

```
sudo nano /etc/subuid
sudo nano /etc/subgid
```

## [](https://dev.to/new#3-add-entries-for-the-user)3\. Add Entries for the User:

Add an entry for the user in both files specifying the UID/GID range. The format is `<username>:<start_uid>:<count>`. For example:

```
&lt;username&gt;:&lt;start_uid&gt;:&lt;count&gt;
```

Replace `<username>` with the actual username, `<start_uid>` with the starting UID, and `<count>` with the number of UIDs/GIDs to allocate. Here’s an example entry:

```
testuser:100000:65536
```

This example allocates `65536` subordinate UIDs/GIDs starting from UID/GID `100000` for the user testuser.

## [](https://dev.to/new#4-save-and-exit)4\. Save and Exit:

Save the changes and exit the text editor.

## [](https://dev.to/new#5-verify)5\. Verify:

Verify that the changes have been applied correctly:

```
grep ^&lt;username&gt; /etc/subuid
grep ^&lt;username&gt; /etc/subgid
```

Replace with the actual username.

By completing these steps, you ensure that the user running Docker in Rootless mode has a sufficient number of subordinate UIDs/GIDs allocated for proper operation.

-   Additional considerations for specific distributions: Depending on the Linux distribution being used, additional steps may be required. For example, on Ubuntu, it’s recommended to use the Ubuntu kernel and install the dbus-user-session package.

It’s recommended to use the Ubuntu kernel for optimal compatibility and performance with Docker in Rootless mode. Ubuntu kernels are typically well-supported and include necessary patches and configurations.

### [](https://dev.to/new#install-the-dbususersession-package)Install the dbus-user-session Package:

On Ubuntu, installing the dbus-user-session package is recommended for proper functioning of certain system services and components. This package provides the D-Bus session bus for user sessions, which can be crucial for running services like Docker.

To install dbus-user-session, you can use the following command:

```
sudo apt-get install dbus-user-session
```

This will ensure that the necessary D-Bus session bus is available for user sessions, which can be essential for applications like Docker to communicate with other system services.

## [](https://dev.to/new#installation-steps)Installation Steps

### With Packages (RPM/DEB)

If you’re using Docker version 20.10 or later and have installed it using RPM or DEB packages, setting up Docker in Rootless mode is straightforward:

Run the below command as a non-root user to set up the daemon.

```
 dockerd-rootless-setuptool.sh install
```

Ensure that the required environment variables (PATH and DOCKER\_HOST) are correctly configured.

### Without Packages

If you’re installing Docker without using packages, you may need to follow slightly different steps:

-   Install the docker-ce-rootless-extras package manually, if necessary.
-   Set up the required environment variables (PATH and DOCKER\_HOST) manually.

If you’re installing Docker without using packages, such as using the binary installation method, you may need to take additional steps to configure Docker in Rootless mode. Here’s how you can do it:

### Install the docker-ce-rootless-extras Package Manually:

The docker-ce-rootless-extras package contains additional tools and configurations specifically for running Docker in Rootless mode. You can download and install this package manually if it’s not included in your installation. Here’s how you can do it:

```
sudo apt-get install -y docker-ce-rootless-extras
```

If you’re using a different package manager or distribution, adjust the command accordingly.

### Set Up Environment Variables (PATH and DOCKER\_HOST) Manually:

After installing the necessary package, you’ll need to set up the required environment variables manually. This ensures that your shell knows where to find the Docker binaries and how to connect to the Docker daemon.

Open your shell configuration file (e.g., ~/.bashrc, ~/.bash\_profile, or ~/.zshrc) in a text editor:

```
nano ~/.bashrc
```

Add the following lines to set up the PATH and DOCKER\_HOST environment variables:

```
export PATH="/usr/bin:$PATH"  # Adjust the path if Docker binaries are located elsewhere
export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock
```

Save the file and exit the text editor. Then, reload your shell configuration to apply the changes:

```
source ~/.bashrc
```

Replace /usr/bin with the actual path to the Docker binaries if it’s different on your system.

By following these steps, you can manually install the necessary Docker package for Rootless mode and configure the required environment variables to ensure proper operation. Make sure to adjust the commands and paths according to your system configuration.

## [](https://dev.to/new#usage-and-best-practices)Usage and Best Practices

Once Docker is configured in Rootless mode, you can manage the daemon and containers using standard Docker commands. However, there are some best practices to keep in mind:

-   Managing Daemon: Use systemctl –user commands to manage the Docker daemon’s lifecycle. Enabling lingering ensures that the daemon starts automatically on system boot.
-   Managing Client: Specify either the socket path or the CLI context explicitly when interacting with the Docker client.
-   Rootless Docker in Docker: To run Rootless Docker inside another Docker container, use the appropriate Docker image and ensure that necessary privileges are granted.
-   Exposing Docker API Socket: If you need to expose the Docker API socket through TCP or SSH, follow the recommended configurations.

## [](https://dev.to/new#troubleshooting)Troubleshooting

In case of errors or unexpected behavior, refer to the troubleshooting section for common issues and their solutions. This includes addressing errors during Docker daemon startup, issues with Docker pull or run commands, networking problems, and more.

## [](https://dev.to/new#conclusion)Conclusion

Running the Docker daemon and containers in Rootless mode offers a more secure alternative to the traditional root-based setup. By following the steps outlined in this guide and adhering to best practices, users can enjoy the benefits of Docker while mitigating potential security risks associated with root access. Whether you’re a developer testing applications locally or a system administrator managing production deployments, Rootless mode provides a safer environment for container operations.



--------------

Ever found yourself in a situation where you want to use Docker but don’t have root access? Or perhaps you wish to provide Docker access to your team or clients but aren’t comfortable giving them root privileges.

The solution to this problem is rootless Docker containers. Read on to learn what it’s all about and how to set it up for yourself.

![](https://www.liquidweb.com/wp-content/uploads/2024/03/image01-Docker-Logo.jpg)

## What Is Rootless Docker?

Rootless Docker installations require only the Docker daemon to run as root, while Docker containers operate as regular Linux users. Docker usually requires root access on the host system, creating a security risk since both the Docker container and the daemon service will operate as root. This means a compromised service operating in a container provides the attacker access to the system files because containers are not truly isolated.

The open source Podman project was the first solution to run containers without root access. Feeling the pressure, Docker added similar functionality, enabling other users to manage Docker containers in a rootless environment. In addition, the rootless installation is available directly from Docker, eliminating the need to utilize Podman.

## How Rootless Docker Works

Many Docker features require you to provide root-level access. Rootless mode circumvents this limitation by utilizing something known as user namespaces.

User namespaces can map user IDs so that the inner namespace root user maps to an unprivileged range in the parent namespace. Since user namespaces have existed for quite a while, this feature has been available in Docker for some time.

The rootless mode operates similarly, except it first creates a user namespace and then starts the daemon in the remapped namespace. The daemon and containers use a different user namespace than the host.

## Prerequisites

-   A Linux server with root-level access.
-   Install _uidmap_ to remap Linux users.

### Configuring the System With _uidmap_ Before Docker Installation

Install _uidmap_ with the following command.

```
apt-get install uidmap -y
```

## How to Install Rootless Docker

Now that the system is ready, we can proceed with installing rootless Docker. It’s easy using the steps outlined below.

### Step 1:

First, download the rootless Docker installation from [their website](https://get.docker.com/rootless) with the following command.

```
curl -sSL https://get.docker.com/rootless | sh
```

\[su\_box title=”Note:” style=”glass” box\_color=”#3ac6eb” radius=”20″\]Piping curl directly to bash is not a best practice and should only be done with trusted sources.\[/su\_box\]

### Step 2:

Once the installation completes, open the _.bashrc_ file with your favorite text editor, as shown below.

```
vi ~/.bashrc
```

### Step 3:

Add the following two environmental variables to the file.

```
export PATH=/home/$user/bin:$PATH
export DOCKER_HOST=unix:///run/user/$id/docker.sock
```

Replace _$user_ with the Linux user you want to install Docker for and replace _$id_ with that user’s ID code. If you don’t know the ID, you can retrieve it using the following command.

```
id
```

### Step 4:

Save the changes you made to the file by pressing the **Esc** key to enter Command mode, and then type **:wq** to write and quit the file.

### Step 5:

Initiate the rootless Docker daemon with the following command, but make sure to replace _$user_ with the proper Linux user.

```
systemctl --$user start docker
```

## How to Configure Rootless Docker

Here are some good practices to consider when using rootless Docker.

### Step 1:Rootless Docker in Regular Docker

To use rootless Docker inside root Docker, use _docker:<version>-dind-_**_rootless_** (not merely _docker:<version>-dind_). See the command below for reference.

```
 docker run -d --name dind-rootless --privileged docker:20.10-dind-rootless
```

### Step 2: Ping Packet Routing

To make ping work, open _/etc/sysctl.conf_ and add the code below.

```
net.ipv4.ping_group_range = 0 2147483647
```

To use ping, run the following command.

```
sudo sysctl --system
```

### Step 3: Expose Privileged Ports

To expose privileged ports, set _CAP\_NET\_BIND\_SERVICE_ on the _rootlesskit_ binary, followed by a restart. See the two commands below.

```
sudo setcap cap_net_bind_service=ep $(which rootlesskit)
systemctl --user restart docker
```

## Use Cases for Running Rootless Docker Containers

Now that you’ve installed and configured rootless Docker, you might wonder what its use cases are.

### Shared Development Environments

Software development teams often share the same server environments, and their understanding of Linux and DevOps practices can vary. Thanks to rootless Docker containers, businesses can install a separate container for their Linux users without impacting other employees using the same server.

### Cost-Saving Measures

Perhaps you’re offering a service that requires your clients to use Docker. Instead of running a separate server instance for each user, you can cut costs by hosting multiple users on the same server with rootless Docker containers.

### Overcome Hosting Limitations

Another good use case for rootless Docker is when your hosting company or hosting plan doesn’t have root access. Provided the support team can install the _uidmap_ package, you can potentially set up rootless Docker yourself.

## Limitations of Rootless Docker

While rootless Docker containers are a great solution with many use cases, there are some limits of which you should be aware.

### Limited Support of Storage Drivers

Rootless Docker only supports the following storage drivers:

-   overlay2
-   fuse-overlayfs
-   btrfs
-   vfs

### Limited Features

The following Docker features are not supported:

-   AppArmor
-   Checkpoint
-   overlay network
-   exposing SCTP ports

### Cgroup Limits

Cgroup is only supported when you run it with systemd and cgroup 2.

## Testing Rootless Docker

After completing the setup, test the rootless Docker container. To do this, follow the steps outlined below.

### Step 1:

To check the proper installation of Docker, issue the following command.

```
docker run hello-world
```

If you get the following message, all is well.

```
Hello from Docker! This message shows that your installation appears to be working correctly.
```

### Step 2:

Now, it’s time to test if Docker works in rootless mode by attempting to run a container with the following command.

```
docker run -it ubuntu bash
```

### Step 3:

Finally, run the following command to verify that the process is running correctly.

```
ps aux
```

## Final Thoughts

This guide helps you know what rootless Docker is and how it works. Installing and running rootless Docker allows developers and administrators to save costs and server resources. Proper testing after installation goes a long way to ensure you are ready to run your containers when needed.

Are you looking for a Linux server for your next Docker-dependent project? Liquid Web offers a variety of [Dedicated Server](https://www.liquidweb.com/dedicated-server-hosting/) solutions to suit your needs. [Contact](https://www.liquidweb.com/contact-us/) our sales team to learn more.
