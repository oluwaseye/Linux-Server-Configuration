# Linux Server Configuration

This project requires a Linux server configured on [Amazon Lightsail](https://aws.amazon.com/lightsail) to serve an application built with Flask, a Python framework.

The web application has been built and can be found here: https://github.com/oluwaseye/catalog

You can visit  [54.196.98.216](http://54.196.98.216/)  for the website deployed.



## 1. Setup LightSail Instance


1.  Create an [AWS Management Console](https://aws.amazon.com/console/) account. 
2.   Create an Amazon Lightsail instance.

![enter image description here](https://i.ibb.co/SvSYChB/instance.png)
*The instance needs at least two minutes to be created and booted up. I named my instance Udacity, you can decided whether or not to name your instance.*

3. Navigate to the account page and download your SSH key

![enter image description here](https://i.ibb.co/KrVrszQ/instance1.png)
 - Click the 'Networking' tab and add the required ports for the project. *(This is an important due to the Lightsail only recognizing port 80 and 22 being set up in the account before applying Ubuntu UFW in the shell. If you skip this step, you will end up locking yourself out of the server)*
![enter image description here](https://i.ibb.co/kDLHhn3/instance2.png)

 - Launch your terminal, navigate to the location of the downloaded SSH key  
 - Rename the file  `sudo mv <filename>.pem udacity_key.pem`
 - Create a ssh directory to store your SSH keys. `mkidir ~/.ssh` and move the file to  `.ssh` directory with  `sudo mv udacity_key.pem ~/.ssh`
 -   Change file permissions with  `sudo chmod 600 ~/.ssh/udacity_key.pem`
*(**Important information**: If you are on a **Windows machine**, `sudo` might not be required while running bash commands to navigate or carried out instruction for your machine. When you login to the **remote machine (Linux)**, then `sudo` will be required in some cases.)*

## 2. Configure your Lightsail Instance

**Create New User Grader and Give Sudo Access**
 - Connect with SSH by entering `sudo ssh -i ~/.ssh/udacity_key.pem ubunutu@54.196.98.216` The ip address is the Public IP of your Lightsail Instance.

 - Type `$ sudo su -` to assume the root user role, then type `$ sudo adduser grader` to create another user **'grader'** This is the account that will be used manage this instance.

 - Create a new file under the sudoers directory: `$ sudo nano /etc/sudoers.d/grader`. Fill that file with `grader ALL=(ALL:ALL) ALL`, then save with (Control X, then type `yes`, then hit the return key on your keyboard)

 - Switch to your newly created account `sudo su grader`

 - Edit the hosts by `$ sudo nano /etc/hosts`, and then add `127.0.0.1 ip-172-26-3-75` under `127.0.0.1:localhost` This will prevent you from encountering `unable to resolve host` error.
   
 - **Update all packages**:

   `$ sudo apt-get update`
  `$ sudo apt-get upgrade`

 - Open another Terminal window  to generate an rsa key for the **grader** user  `$sudo ssh-keygen -f ~/.ssh/udacity_key.rsa`

 - Output the public key in the new terminal window, `$ sudo cat ~/.ssh/udacity_key.rsa.pub` , select and copy the key.

 - Back to the previous terminal window where you are logged into Amazon Lightsail, navigate to grader's folder by  `$ cd /home/grader`

 - Create a .ssh directory:  `$ mkdir .ssh`

 - Create a file to store the public key:  `$ touch .ssh/authorized_keys`
   
 - Edit the authorized_keys file  `$ nano .ssh/authorized_keys` and paste the public key copied from the other terminal into the new  `authorized_keys` file, save and exit.

 - Change the permission:  `$ sudo chmod 700 /home/grader/.ssh`  and  `$
   sudo chmod 644 /home/grader/.ssh/authorized_keys`

    

 - Change the owner from root to grader:  `$ sudo chown -R grader:grader
   /home/grader/.ssh`

 - We now need to enforce the **key-based authentication** and change the **SSH port from 22 to 220**:  `$ sudo nano /etc/ssh/sshd_config`. Update the *Port* to `2200` , also Locate *PasswordAuthentication* and
   change options to  `no` ***if it says otherwise. I did not have to change this line since the file already had no. This might be different for you.***  Restart ssh :  `$ sudo service ssh restart`
    
Disconnect the server by  typing `$ logout` and log back in through port 2200:  `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@54.196.98.216`
    


## 3. Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for **SSH (port 2200)**, **HTTP (port 80)**, and **NTP (port 123)**

```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable 

```
To check the configuration, type `sudo ufw status`. The output must look like this:
```
To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
```

## 4. Configure the local timezone to UTC

  Configure the time zone  `sudo dpkg-reconfigure tzdata` A set of instruction will appear for you to follow. Press the enter to select **None of the above** option and select the **UTC option**.


## 5. Install Apache,  Python as a mod_wsgi application, and Git

 - Install Apache  `sudo apt-get install apache2`
 -  Install mod_wsgi and python `sudo apt-get install libapache2-mod-wsgi python-dev`
 -  Install Git  `sudo apt-get install git`
 - Restart apache `sudo service apache2 restart`