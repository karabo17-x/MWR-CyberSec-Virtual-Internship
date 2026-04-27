#Completing the Assignments 


 you will need a virtual machine to complete some of the weekly assignments. While the TryHackMe AttackBox is usable, it is limited to 1 hour per day for a free user.
 If your device cannot handle a Virtual machine, you will need to use that 1 hour per day and complete the assignments over multiple days of the week. I do recommend the use of a VM if you can use one. 


#Completing Of Tasks and Assigments.

TryHackMe has a VPN that allows unlimited connectivity to it which removes that time limit. You should connect to that VPN from a Kali Linux (https://www.kali.org/get-kali/#kali-platforms) Virtual Machine. 


There is a guide for this from Kali - https://www.kali.org/docs/virtualization/install-virtualbox-guest-vm/  


To Set Up a Virtual Machine perform the following steps: 

    Download a Virtualisation platform like VirtualBox  (https://www.virtualbox.org/) 
    Install VirtualBox
    Download the VirtualBox machine image from Kali (https://www.kali.org/get-kali/#kali-virtual-machines )
    Unzip the downloaded file, you may need 7zip or equivalent for this.
    Open VirtualBox
        In the top left of VirtualBox, Click Machine -> Add
    Select the kali-linux.vbox file (File name is subject to change, but it will have th .vbox extension)
    Select the machine and boot it 

 

If the above doesn’t work, you can also install kali into VirtualBox 

    Download a Virtualisation platform like VirtualBox  (https://www.virtualbox.org/) 
    Install VirtualBox
    Download the kali Installer (.iso) file from https://www.kali.org/get-kali/#kali-installer-images 
    Open VirtualBox
        In the top left of VirtualBox, Click Machine -> New
    Give the machine a good name
    Select the .iso file you downloaded for “ISO Image”
    Go to hardware, the more base memory and processors you add, the more smoothly the machine should run. It is a slider, so don’t go into the red as that may be too much for your machine
    Click finish
    Boot the Machine and follow the installation instructions. Here is a guide from kali if needed - https://www.kali.org/docs/installation/hard-disk-install/  

 

BurpSuite 

BurpSuite comes installed with Kali Linux, if you do want to set it up yourself, PortSwigger has a decent guide here - https://portswigger.net/burp/documentation/desktop/getting-started/download-and-install  


