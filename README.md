# JetBrains Projector with Android Studio (On a local Ubuntu/Linux machine)

Guide to setup JetBrains Projector and access Android Studio from any device.

[Blog post](https://joenrv.medium.com/how-to-run-android-studio-on-any-device-with-jetbrains-projector-3d9d23a8c179)

![Android Studio on iPad Pro](ipad.jpg)
*Android Studio on an iPad Pro*

### Step 1: Set up your host (the local powerhouse that will run the builds)

1. Make sure the machine is fully set up and (local) network connectivity is available
2. Enable SSH connectivity: https://linuxize.com/post/how-to-enable-ssh-on-ubuntu-18-04/

### Step 2: Generate SSH keys

1. Use your available or generate new SSH keys: https://linuxize.com/post/how-to-set-up-ssh-keys-on-ubuntu-1804/ (just make sure your have something like local `~/.ssh/id_rsa` & `~/.ssh/id_rsa.pub` files)
2. Copy the SSH key over to your host machine: `$ ssh-copy-id myuser@myhostname.local`

### Step 2: Connect to your host via SSH

1. Modify or create the ~/.ssh/config file on your client (the device you will access the host by)
2. Add a new host-configuration to your ~/.ssh/config:
```
Host {REMOTE_MACHINE_ALIAS}
  User {REMOTE_MACHINE_USERNAME}
  HostName {REMOTE_MACHINE_IP_OR_HOSTNAME}
  Port 22
  IdentityFile ~/.ssh/{SSH_KEY_NAME} (SSH_KEY_NAME is likely id_rsa)
  PreferredAuthentications publickey
  ControlMaster auto
  ControlPath /tmp/%r@%h:%p
  ControlPersist 1h
```
* `Host` - Choose an alias like `powerhouse` or anything you like
* `User` - The username you've set up on your host (f.e.: myuser)
* `Hostname` - The hostname set up on your Ubuntu machine with `.local` suffix (f.e: myhostname.local)
* `IdentityFile` - Path to the SSH file on your client device

3. Now you can connect to your machine easily like this

```
$ ssh powerhouse
```
4. Once connected, you can now begin installing things on your remote server

### Step 3: Install Projector and Android Studio on the remote server

1. To install Projector (and dependencies), run the following commands:
```
$ sudo apt install python3 python3-pip
$ sudo apt install libxext6 libxrender1 libxtst6 libfreetype6 libxi6
$ pip3 install projector-installer
```
2. Projector is now installed, you can see all options by running 
```
projector --help
```
Should this command fail, it's possible you still need to add the bin directory to your PATH on your host machine. Modify the `~/.bashrc` or `~/.zshrc` file on your host machine and add the following line somewhere on the bottom:

```
$ export PATH=$PATH:/home/sigmadelta/.local/bin
```

3. At this point, you could run one of the pre-configured IDEs like IntelliJ and start using it, but to run Android Studio, you need to install it separately.
4. Download the latest Android Studio 4.2 or above (4.2 is the minimum version that works with Projector)
* Find the latest download URL for linux from https://developer.android.com/studio/archive. At the time of writing, the latest version is 4.2 Canary 16
* Download it to your remote server in your home directory with curl: 

```
$ curl "https://r1---sn-5hnedn7z.gvt1.com/edgedl/android/studio/ide-zips/4.2.0.16/android-studio-ide-202.6939830-linux.tar.gz?mh=gV&mvi=1&pl=33&shardbypass=yes&redirect_counter=1&cm2rm=sn-uxaxoxu-cg0r7e&req_id=29c47d8c52741f82&cms_redirect=yes&ipbypass=yes&mip=2a02:1810:4d63:4600:a17d:37fb:9abd:d96f&mm=42&mn=sn-5hnedn7z&ms=onc&mt=1605264385&mv=m" --output android-studio.tar.gz
```
* Unzip the downloaded archive:

```
$ tar -xvf android-studio.tar.gz
```
5. You now have Android Studio installed, all that is left is to configure Projector, making sure to select the port that was chosen as the custom TCP rule for the VM (step 1.6) - here we're using port 8888
```
$ projector config add
Enter a new configuration name: AndroidStudio
Do you want to choose a Projector-installed IDE? [y/n]: n
Enter the path to IDE: /path/to/your/android-studio
Enter a desired Projector port (press ENTER for default) [10005]: 8888
Use secure connection (this option requires installing a projector's certificate to browser)? [y/n]: y (** You will need this if you want to make use of the clipboard to copy things from your host machine to your local clipboard when f.e. wanting to Google an error **)
Would you like to set password for connection? [y/n]: n
```

#### Certificate error
During the creation of the certificate file, I got an error due to the fact that the projector script could not find the keytool inside of `android-studio/jbr/bin`. I've created a ticket to follow this up for now: https://youtrack.jetbrains.com/issue/PRJ-191


### Step 4: Access Android Studio from a Browser

1. On your local machine, start a browser and go to `http://<your_server_ip>:8888`
2. Click OK on the dialog that shows up
3. Enjoy your remote Android Studio!

### Optional: Setup ADB to deploy to / debug a local device

1. You need to have the same version of adb on the remote and local machine
```
$ adb -- version
Android Debug Bridge version 1.0.41
Version 30.0.4-6686687
```
2. Kill adb on both local and remote machines if it was already running
```
$ adb kill-server
```
3. On the local machine, with an Android device connected, run:
```
$ adb devices
List of devices attached
  ABCDEF12345
$ ssh -R 5037:localhost:5037 powerhouse
```
4. This will connect to your instance with port forwarding enabled. Now check that the device is visible on the remote machine:
```
$ adb devices
List of devices attached
  ABCDEF12345
```
5. That's it! Both machines can now run adb commands and everything will be redirected to the local phone.
6. To always connect to your instance with adb port forwarding (port 5037 by default), you can add the following line to your `~/.ssh/confg`:
```
RemoteForward 5037 localhost:5037
```


### Useful links and other installation methods

[Main Projector README](https://github.com/JetBrains/projector-server/blob/master/README-JETBRAINS.md)

[Projector Docker Image](https://github.com/JetBrains/projector-docker)

[Projector Installer Repo](https://github.com/JetBrains/projector-installer)

[Projecter Server Repo](https://github.com/JetBrains/projector-server/blob/master/docs/Projector.md)

