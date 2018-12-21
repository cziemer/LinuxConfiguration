The Following are the steps documenting the summary of configurations made, a list of third party resources used to complete the project and the information required to connect.
https://review.udacity.com/#!/projects/3573679011/rubric
## Server Details
* Public IP: 18.213.218.50 Port: 2200
* Application URL: [http://18.213.218.50.xip.io.](http://18.213.218.50.xip.io)
* Hosting by: Amazon Lightsail
* Username / Password: grader, grader

## Summary of Software Installed
* Apache2
* postgreSQL
* mod_wsgi
* Python-pip
* virtualEnv
* Flask
* sqlAlchemy
* oauth2client
* httplib2
* requests

## 1. Get a Server setup
1. Log into (or create an account) for [Amazon Lightsail](https://lightsail.aws.amazon.com/).
2. Create an instance
3. Set image as Ubuntu
4. Setup a hostname and await startup
5. Download the private key from Amazon
6. SSH to server with private key `ssh -i ~/.ssh/LightsailDefaultKey.pem ubuntu@18.213.218.50 -p 22` or use link available on AWS site to connect

## 2. Secure the Server
1. Update all currently installed packages
   * Get a list of all available updates `sudo apt-get update`
   * Update all packages where available `sudo apt-get upgrade`
2. Change the SSH port from **22** to **2200**
   * Edit the config file `sudo nano /etc/ssh/sshd_config`
   * Manually edit port 22 to 2200 (toward top of file)
   * Save (Write Out) the update
   * Restart the SSH to take new configuration `sudo service ssh restart`
3. Configure UFW (Firewall) to only allow incoming connections for SSH (port 2200), HTTP (port 80) and NTP (port 123)
   * Check current firewall status `sudo ufw status` **Note**: Will show Status: inactive if not yet configured
   * Block all incoming ports by default `sudo ufw default deny incoming`
   * Allow all outgoing ports `sudo ufw default allow outgoing`
   * Allow Incoming Connection on SSH Port (2200) `sudo ufw allow 2200/tcp`
   * Allow www (port 80) 'sudo ufw allow www'
   * Allow Incoming Connection for NTP port (123) `sudo ufw allow 123/udp`
   * Enable the Firewall (be sure that SSH access is allowed) `sudo ufw enable`
   * Modify Lightsail Instance settings to allow the above changes and remove SSH port 22
   * Verify Firewall settings `sudo ufw status`
   

| To | Action | From |
|-----------|------------|-----------|
|2200/tcp|ALLOW|Anywhere|
|80/tcp|ALLOW|Anywhere|
123/udp|ALLOW|Anywhere|
2200/tcp (v6)|ALLOW|Anywhere (v6)|
80/tcp (v6)|ALLOW|Anywhere (v6)|
123/udp (v6)|ALLOW|Anywhere (v6)|


## 3. Give *grader* access
1. Create a new user account named grader `sudo adduser grader`
2. Give grader permission to sudo
   * Check for current sudo users `sudo ls /etc/sudoers.d`
   * Add new file under sudoers directory `sudo nano /etc/sudoers.d/grader`
   * Inside the new file, add the line `grader ALL=(ALL:ALL) ALL` to give sudo access
   * Check sudo users again to confirm grader is listed `sudo ls /etc/sudoers.dsudo ls /etc/sudoers.d`
3. Create an SSH key pair for grader
   * Open a local virtual machine and generate a new key `ssh-keygen`
   * Copy the new key by entering `cat .ssh/grader_key.pub`
   * Back on the server terminal, switch to the grader users `sudo -i -u grader`
   * Create .ssh directory `mkdir .ssh`
   * Create authorized_keys file in .ssh `touch .ssh/authorized_keys`
   * Open the new file and paste in the key from above, then save the file `nano .ssh/authorized_keys`
   * Setup permissions on file and directory
     * `sudo chmod 700 /home/grader/.ssh`
     * `sudo chmod 644 /home/grader/.ssh/authorized_keys`
## 4. Prepare to deploy the project
1. Configure the local timezone to UTC `sudo dpkg-reconfigure tzdata`
   * For Geographic Area, select *None of the above*
   * For Time zone, select *UTC*
2. Install and configure Apache to serve  a Python mod_wsgi application
   * Install Apache2 `sudo apt-get install apache2`
   * Install mod_wsgi `libapache2-mod-wsgi`
   * Install Python-pip `sudo apt-get install python-pip`
   * Install VirtualEnv `sudo pip install virtualenv`
   * Start the Virtual Environment `sudo virtualenv venv` `source venv/bin/activate`
   * Install Flask and other packages `sudo pip install Flask sqlalchemy oauth2client httplib2 requests`
3. Install and configure PostgreSQL `sudo apt-get install postgresql postgresql-contrib`
   * Change to the SuperUser `sudo -i -u postgres`
   * Create a new role `createuser --interactive`
    * Name: catalog
    * superuser: No
    * Create Databases: Yes
    * Create more new roles: No
   * Give the new user permission to create databases `ALTER USER catalog CREATEDB;`
   * Create the new database `createdb catalog`
   * Logout of the superuser account, then add catalog user `sudo adduser catalog`
     * Added password *catalog*
   * Connect to the new database `\c catalog`
   * Revoke all rights to the database `REVOKE ALL ON SCHEMA public FROM public;`
   * Only allow catalog to create tables `GRANT ALL ON SCHEMA public TO catalog;`
   * Go into PSQL to check permissions
     * `sudo -i -u catalog`
     * `psql`
     * `\du`
     *
| Role Name | Attributes | Member of |
|-----------|------------|-----------|
| catalog   | Create DB  | {}        |
| postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}        |

   * Do not allow remote connections
     * This is current default when installing PostgreSQL from the Ubuntu properties [source](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
4. Install GIT `sudo service apache2 start` `sudo apt-get install git`

## 5. Deploy the Item Catalog project
1. Clone and setup Item Catalog project from Github
   * Change Directory back to www `cd /var/www`
   * Create a Catalog directory to store the files `sudo mkdir catalog`
   * Make Grader the owner `sudo chown -R grader:grader catalog`
   * Go to the Catalog Directory `cd catalog`
   * Clone the GIT Repo `git clone https://github.com/cziemer/FullStackItemCatelog` (yes, it was misspelled)
2. Minor changes to project files due to running on a server versus local server:
   * Rename finalproject.py to \__init__.py `finalproject.py __init__.py`
   * Reconfigure oAuth on Google
   * Update client_secrets file with new info from Google
   * Update the path of client_secrets inside \__init__.py
   * Add app secret key for the flask app in \__init__.py
   * Update path of python engine in database_setup.py and application.py
     * Open the file `sudo nano database_setup.py` (and application.py)
     * Update line stating with *Engine* to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
3. Build the database `python database_setup.py`
4. Populate the Database `python manycars.py`
5. Ensure .git directory is not publicly accessible
   * Create .htaccess file `cd /var/www/catalog/`
   * Edit above file `sudo nano .htaccess`
   * Paste code in file and save `RedirectMatch 404 /\.git`
## 5. Additional Requirements
1. Block root remote access & Block passwords
   * Open config file `sudo nano /etc/ssh/sshd_config`
     * Change *PasswordAuthentication* to *no* (Was already set)
     * Change *PermitRootLogin* to *no*
     * Save File
   * Restart the server to enforce the changes `sudo service ssh restart`
2. Authentication via key-based RSA only (see above)

## Sources Consulted

* [Digital Ocean - How to Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Disable SSH login from a server](https://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server)
* [How to Install and Use PostgreSQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
* [Pip and Virtualenv for Beginners](https://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/)
* [Creating a Server with Amazon Lightsail](https://medium.com/@mariasurmenok/creating-a-server-with-amazon-lightsail-11c377cf814c)
* [Check Remote Database Connection permissions](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [Prevent access to git directory](https://serverfault.com/questions/128069/how-do-i-prevent-apache-from-serving-the-git-directory)
* [Student Project Consulted for configuration settings](https://libraries.io/github/golgtwins/Udacity-P7-Linux-Server-Configuration)