# How to use Trezor on Qubes OS
Written by Ursidae: https://ursidaecyber.com

This repostiory will guide the process of installing Trezor Suite and using Trezor hardware wallets on Qubes OS.
Unfortunately installing Trezor Suite is not as straight forward as installing other software on Qubes is. So let’s dive in.
This guide contains two parts: brief instructions and in-depth instructions. Use whichever is suited to your needs.

Link to in-depth instructions if you require detail: https://github.com/UrsidaeCyber/Trezor-Qubes#in-depth-instructions

# Brief Instructions:

## Step 1: Install Trezor Suite

1. Install the Trezor Suite .AppImage from the Trezor website along with the signature and signing key in a new Whonix AppVM dedicated to Trezor.

2. Verify the download.

3. Execute in terminal: `sudo chmod u+x /Downloads/Trezor-Suite-23.4.2-linux-x86_64.AppImage`

4. Right click on the .AppImage file and press execute to open the application.

## Step 2: Port Listening

In Trezor Whonix AppVM:

1. `sudo nano /rw/config/rc.local`

2. Add the following code to the file:

`socat TCP-LISTEN:21325,fork EXEC:”qrexec-client-vm sys-usb trezord-service” &`

3. Save and exit.

## Step 3: Dom0 Trezor Policy

In Dom0:

1. `sudo nano /etc/qubes-rpc/policy/trezord-service`

2. Add this code to the new file:

`$anyvm $anyvm allow,user=trezord,target=sys-usb`

3. Save and exit.

## Step 4: Fedora Cloning

1. Clone your current regular fedora-37 template Qube and name it fedora-37-sys.

2. Clone the fedora-37-dvm Qube and name it fedora-37-sys-dvm.

3. Set the template for the fedora-37-sys-dvm as fedora-37-sys.

4. Set sys-usb’s template as fedora-37-sys-dvm.

## Step 5: Trezord Service

In fedora-37-sys-dvm:

1. `sudo mkdir /usr/local/etc/qubes-rpc`

2. `sudo nano /usr/local/etc/qubes-rpc/trezord-service`

3. Add this code to the file:

`socat - TCP:localhost:21325`

4. Save and exit.

5. `sudo chmod +x /usr/local/etc/qubes-rpc/trezord-service`

## Step 6: Trezor Bridge

In fedora-37-sys:

Download the Trezor Bridge .rpm file from Trezor.

1. `sudo chmod u+x /Downloads/trezor-bridge-2.0.27-1.x86_64.rpm`

2. `sudo rpm -i /Downloads/trezor-bridge-2.0.27-1.x86_64.rpm`

## Step 7: Udev rules

Note on Udev rpm use: Using the Trezor-provided Udev rpm file does not work for Qubes. See in-depth explanation section below. Use the provided Method 1 or 2 here. Use method 1 if comforable with enabling networking in template and method 2 if not.

### Method 1: Manual Build

In fedora-37-sys:

1. `sudo nano /etc/udev/rules.d/51-trezor.rules`

Copy and paste this code into the file:

```
# Trezor

SUBSYSTEM=="usb", ATTR{idVendor}=="534c", ATTR{idProduct}=="0001", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl", SYMLINK+="trezor%n"

KERNEL=="hidraw*", ATTRS{idVendor}=="534c", ATTRS{idProduct}=="0001", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl"

# Trezor v2

SUBSYSTEM=="usb", ATTR{idVendor}=="1209", ATTR{idProduct}=="53c0", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl", SYMLINK+="trezor%n"

SUBSYSTEM=="usb", ATTR{idVendor}=="1209", ATTR{idProduct}=="53c1", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl", SYMLINK+="trezor%n"

KERNEL=="hidraw*", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="53c1", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl"
```

2. Save and exit.

3. `sudo chmod +x /etc/udev/rules.d/51-trezor.rules`

### OR

### Method 2: Curl Installation

1. In fedora-37-sys enable networking.

2. `sudo dnf install curl`

3. `sudo curl https://data.trezor.io/udev/51-trezor.rules -o /etc/udev/rules.d/51-trezor.rules`

4. `sudo chmod +x /etc/udev/rules.d/51-trezor.rules`

5. Revoke fedora-37-sys networking permissions.

## Step 8: Install Trezor Dependencies

In the Trezor Whonix AppVM:

1. `sudo apt install pip`

2. `pip3 install --user trezor`

### AND

In fedora-37-sys:

1. Allow networking.

2.  `sudo dnf install trezor-common`

3. Revoke networking permissions in fedora-37-sys.

# In-Depth Instructions:

## Step 1: Install Trezor Suite

1. Create a new Whonix template specifically for cryptocurrency use. We will install some programs at the template level and may not want these to potentially interfere with our regular Whonix use.

2. Create a Whonix AppVM based on your new Crypto Whonix template which you will now use Trezor on.

3. Navigate to Trezor’s website and download the Trezor suite application. It will come in a .AppImage file for Linux. Download the signature and 2021 signing key along with it. Verify the download.

4. Open a terminal window in your Whonix AppVM and run the following command to allow the Trezor Suite .AppImage to be executed as a program:

`sudo chmod u+x /Downloads/Trezor-Suite-23.4.2-linux-x86_64.AppImage`

Make sure to adjust the command text to account for the name of your file as it may not be the same depending on the current version. Remember to do this for all subsequent steps.

You can open the application by right-clicking on the .AppImage file and clicking “execute”.

## Step 2: Port Listening

1. Start a Terminal window in your new Trezor-dedicated AppVM and execute the following code to edit the rc.local file:

`sudo nano /rw/config/rc.local`

You are now editing the rc.local plain text file through terminal.

2. Navigate to the bottom of the file using your arrow keys and type the following code (note the & at the end):

`socat TCP-LISTEN:21325,fork EXEC:”qrexec-client-vm sys-usb trezord-service” &`

Press Ctrl + X to save.

Press Y to confirm.

Press Enter to exit.

Although this portion of code can be executed in any AppVM with networking, I advise it be done in the AppVM you are dedicating to Trezor Suite to avoid unwanted code elsewhere.

## Step 3: Dom0 Trezor Policy

1. Open terminal in dom0 and run the following code:
`
sudo nano /etc/qubes-rpc/policy/trezord-service`

This will create a plain text file in dom0 within that directory. You are now editing that file in your terminal window.

2. Paste the following code into the file via terminal:

`$anyvm $anyvm allow,user=trezord,target=sys-usb `

3. Press Ctrl + X to save.

Press Y to confirm.

Press Enter to exit.

## Step 4: Fedora Templates

Because sys-usb is a disposable VM, these installations need to be applied to sys-usb’s template as well as the Disposable VM Template’s template in order to maintain persistence. This is also due to how the qrexec function works in Qubes, which is beyond the scope of this guide.

As a security precaution, I recommend not using your default Fedora template for Trezor-use. It is used for Qubes management and all your other Fedora Qubes; we will also be granting networking permissions to the template later in this guide. To remedy this we will dedicate a Fedora template exclusively for Trezor use.

We will be installing these items into both your new Fedora TemplateVM and DisposableVM Template. Pay close attention to which we are working in for each step.

I am currently running Fedora 37, so my Fedora Qube template VM is named fedora-37. Yours will differ depending on your version of Fedora. Adjust the name accordingly.

1. Clone your current regular fedora-37 template Qube and name it fedora-37-sys.

2. Clone the fedora-37-dvm Qube and name it fedora-37-sys-dvm.

3. Set the template for the fedora-37-sys-dvm as fedora-37-sys.

4. Set sys-usb’s template as fedora-37-sys-dvm.

## Step 5: Trezord Service

In fedora-37-sys-dvm:

1. Open terminal and execute the following code:

`sudo mkdir /usr/local/etc/qubes-rpc`

This will create a folder titled qubes-rpc within the specified directory

2. Create a plain text file within that folder titled trezord-service by executing the following code:

`sudo nano /usr/local/etc/qubes-rpc/trezord-service`

3. You are now editing the plain text file within the terminal window. Add the following line of code to the file:

`socat – TCP:localhost:21325 `

Press Ctrl + X to save.

Press Y to confirm then press Enter to exit.

4. Make the new file executable with the following command:

`sudo chmod +x /usr/local/etc/qubes-rpc/trezord-service`

## Step 6: Trezor Bridge

In fedora-37-sys:

1. Download the Trezor Bridge .rpm file from here in a disposable Whonix VM.

2. Open a terminal window in fedora-37-sys and run the following code to allow the rpm file to be executable:

`sudo chmod u+x /Downloads/trezor-bridge-2.0.27-1.x86_64.rpm`

3. Install the Trezor bridge rpm file with the following code:

`sudo rpm -i /Downloads/trezor-bridge-2.0.27-1.x86_64.rpm`

This will automatically install the bridge in the following directories:

/etc/systemd/system/multi-user.target.wants/trezord.service

/usr/lib/systemd/system/trezord.service

Successful installation should result in terminal showing that both directories are communicating with each other: 

> “created symlink /etc/systemd/system/multi-user.target.wants/trezord.service → /usr/lib/systemd/system/trezord.service.”

## Step 7: Udev rules

Installing the Udev rules will create a plain text file titled 51-trezor.rules in /etc/udev/rules.d which will allow dom0 to facilitate communication between sys-usb and your Trezor Qube in the form of an RPC call.

You can read the Trezor Udev rules code on the Github page here.

There are two methods of installing the Udev rules on Qubes. Method 1 (manually building Udev rules) is more secure and equally easy while method 2 (curl installation) introduces some but negligible security risk as it requires the template to be granted networking permission.

It should be noted that the “security risks” that are implied in method 2 are redundant since we are granting networking permission to an isolated template just for Trezor — which is not connected to any other Qubes (other than sys-usb) — and we will revoke networking permissions right after installation. Nonetheless to maximize security, I suggest method 1 as the difficulty level is equal between both.

Note on Udev installation via rpm file:

An alternative option exists to use the Udev rpm file provided by Trezor and run it as an executable. Unfortunately this method does not work on Qubes due to how qrexec sets up inter-domain communication and how templates and AppVMs inherit data. Details are beyond the scope of this guide. If you do not want to give the fedora-37-sys Qube template networking permissions, or for some reason want to use an alternative method, you can build the Udev rules yourself.

If you run the rpm in fedora-37-sys it won’t install in the right directory, and if you run it in fedora-37-sys-dvm, it will install in the right place but won’t remain persistent into sys-usb.

### Method 1: Manual Build

In fedora-37-sys:

1. Run the following code in a terminal window to create the 51-trezor.rules file in the Udev rules directory.

`sudo nano /etc/udev/rules.d/51-trezor.rules`

2. After running this command you are now editing the plain text file you have just created. Copy and paste the following code into terminal:

```
# Trezor

SUBSYSTEM=="usb", ATTR{idVendor}=="534c", ATTR{idProduct}=="0001", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl", SYMLINK+="trezor%n"

KERNEL=="hidraw*", ATTRS{idVendor}=="534c", ATTRS{idProduct}=="0001", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl"

# Trezor v2

SUBSYSTEM=="usb", ATTR{idVendor}=="1209", ATTR{idProduct}=="53c0", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl", SYMLINK+="trezor%n"

SUBSYSTEM=="usb", ATTR{idVendor}=="1209", ATTR{idProduct}=="53c1", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl", SYMLINK+="trezor%n"

KERNEL=="hidraw*", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="53c1", MODE="0660", GROUP="plugdev", TAG+="uaccess", TAG+="udev-acl"
```

3. Press control + X to save the file.

Press Y to confirm.

Press Enter to exit.

4. Make the Udev rules file executable by running the following code in terminal:

`sudo chmod +x /etc/udev/rules.d/51-trezor.rules `

### Method 2: Direct Installation Via Curl

1. Grant networking permissions to fedora-37-sys in the Qubes manager.

2. In a fedora-37-sys terminal window install curl with this command:

`sudo dnf install curl`

3. Download the Trezor Udev rules via curl:

`sudo curl https://data.trezor.io/udev/51-trezor.rules -o /etc/udev/rules.d/51-trezor.rules`

4. Once the Udev rules are installed, verify they are in the right directory. They should be in /etc/udev/rules.d as a file titled 51-trezor.rules.

5. Once you have successfully installed the Udev file in the correct location, make the file executable by using the following command:

`sudo chmod +x /etc/udev/rules.d/51-trezor.rules`

6. Revoke fedora-37-sys networking permissions in the Qubes manager.

## Step 8: Install Trezor Dependencies

1. In the Whonix AppVM where you are using Trezor open a terminal window.

2. Run the following command to install pip. 

`sudo apt install pip`

3. Run the following command to install the trezor package:

`pip3 install --user trezor`

AND

1. Enable networking permissions for fedora-37-sys in the Qubes manager.

2. Run the following command to install the trezor-common package:

`sudo dnf install trezor-common`

3. Revoke fedora-37-sys networking permissions in the Qubes manager.`
