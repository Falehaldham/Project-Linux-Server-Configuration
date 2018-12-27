# Deploying "My Favorite Manga Item Catalog Project" into Lightsail
After delivering my Item Catalog project for the Full Stack Web Developer class and responding to the 3rd project "Linux Server Configuration", I am explaining here the steps I took to successfully deploy my Item Catalog project into Lightsail AWS.

## (req.i) IP address and SSH port so your server can be accessed by the reviewer
---
Static IP: 52.57.211.236
SSH port: 2200
this command can be used to connect to the server:
`ssh grader@52.57.211.236 -p 2200 -i grade_key`
the grade_key is uploaded with the submission of this project.

## (req.ii) complete URL to your hosted web application
---
URL: [My Favorite Manga](http://52.57.211.236.xip.io)

## (req.iii) A summary of software you installed and configuration changes made
---
I have used Amazon Lightsail Linux server to deploy my Item Catalog project. this is a fresh installation of Linux so I had to do some installation and make some configurations to get it to work.
## 1. Creating the server
* I signed up to Amazon Web Service
* I created a new instance and selected the following options:
  * Instance image: OS only, Ubuntu 16.04
  * Instance plan: the lowest was chosen
  * Instance name: Ubuntu-512MB-udacity
* Configured the IP to be static as: 52.57.211.236
* I created an ssh default key for the master user to login into the server from a local terminal
* I configured the instance so it allows for port 2200 in the port tab (custom port)
* I logged in into the server as ubuntu user using this command `ssh ubuntu@52.57.211.236 -p 2200 -i LightsailDefaultKey-eu-central.pem`
## 2. updating and upgrading the server
* I updated all packages at the server using this command: `sudo apt-get update`
* I upgraded all packages at the server using this command: `sudo apt-get upgrade`
* I removed any packages that are not needed using this commend `sudo apt-get autoremove`
## 3. Creating a new user named grader
* added a new user (grader) to my instance using this commend: `sudo adduser grader`
* I gave the newly added user a sudo permission to perform administrative tasks using the following steps:
  * I created a file in this path `/etc/sudoers.d/` called grader using this commend `sudo touch /etc/sudoers.d/ grader`
  * I edited the file `grader` using this commend `sudo nano /etc/sudoers.d/ grader` to include the following line `grader ALL=(ALL) NOPASSWD:ALL`
* now the user `grader` has admin permission and can login into the server using this command `ssh grader@52.57.211.236 -p 2200`
* On my local machine, I have generated a key pair using `ssh-gen` commend and named it `grade_key`
* I stored the grade_key in my local .ssh folder
* I ran this commend on my local .ssh folder `sudo 700 chmod ~/.ssh` to make sure only the owner has full permission on the folder .ssh
* I then logged into my instance using the ubuntu user to configure the permission for the grader user
* I switched to the grader user using su command (`su grader`)
* I created a file called authorized_keys inside a .ssh directory
* I copied the public key content from my local machine to the authorized_keys file
* I ran this command `chmod 700 .ssh` to make sure the .ssh folder is only accessible to the owner (grader) and this commend `chmod 644 .ssh/authorized_keys`
* now the grader user can access the server using this command `ssh grader@52.57.211.236 -p 2200 -i grade_key`
## 4. Securing the Server
* I previously configured the instance so it allows porting through 2200 (see point 1 the last point)
* from within the server, I have changed the SSH port from 22 to 2200 by editing the file called sshd_config in this path `/etc/ssh/sshd_config` using this commend `sudo nano /etc/ssh/sshd_config`
* I then restarted the ssh service using this commend `sudo /etc/init.d/ssh restart`
* to be able to secure the Uncomplicated Firewall (UFW) following the instructions provided for this project, I did the following steps:
  * block all incoming requests: `sudo ufw default deny incoming`
  * allow outgoing connections: `sudo ufw default allow outgoing`
  * allow 2200 port: `sudo ufw allow 2200/tcp`
  * allow www on port 80: `sudo ufw allow www`
  * allow NTP on port 123: `sudo ufw allow ntp`
  * enabling new configuration on the firewall: `sudo ufw enable`
## 5. Disabling the root and changing the time zone
* I accessed the file called sshd_config and made a change to the PermitRootLogin like so:
  * `sudo nano /etc/ssh/sshd_config`
  * changed PermitRootLogin from yes to no: `PermitRootLogin to no`
  * `sudo /etc/init.d/ssh restart`
* I used this commend `sudo timedatectl set-timezone UTC` to change the time zone to UTC
## 6. Prepare to deploy your project
* I have installed the apache web server using the following command:
  * `sudo apt install apache2`
* I installed the XX using this commend:
  * `sudo apt-get install libapache2-mod-wsgi`
* I created an apache configuration file named 'ItemCatalog.conf'
* I configured the apache to handle requests using the WSGI module by editing the ItemCatalog.conf to include the following: 
  ```
  <VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname, and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName 

        ServerAdmin aldham.faleh@gmail.com

        # Define WSGI parameters. The daemon process runs as the www-data user.

        WSGIDaemonProcess catalog user=www-data group=www-data threads=5
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}

        # Define the location of the app's WSGI file
        WSGIScriptAlias / /var/www/ItemCatalog/itemsCatalog.wsgi

        # Allow Apache to serve the WSGI app from the catalog app directory
        <Directory /var/www/ItemCatalog/>
                Require all granted
        </Directory>

        # Setup the static directory (contains CSS, Javascript, etc.)
        Alias /static /var/www/ItemCatalog/static

        # Allow Apache to serve the files from the static directory
        <Directory  /var/www/ItemCatalog/static/>
                Require all granted
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log

        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
* for testing purposes, I created a flask starter code in this path `/var/www/ItemCatalog` and called it app.py. this starter code contains the following:
  ``` python
  # importing Flask framework
  from flask import Flask
  app = Flask(__name__)
  @app.route("/")
  def hello():
      return "Hello, Flask!"
  if __name__ == "__main__":
      app.run()
  ```
* then I restarted apache using this commend:
  * `sudo apache2ctl restart`
* Once the code ran successfully, I renamed my tested app.py to app_test.py and moved to deploy the Item Catalog project to my instance
* for bug capturing, I always used this command (here and in the deployment step):
  * `sudo tail -f /var/log/apache2/error.log`
## 7. Deploy the Item Catalog project
* I used git to clone my project from my github account [GitHub - Falehaldham/ItemCatalog](https://github.com/Falehaldham/ItemCatalog) to this folder `/var/www/ItemCatalog` in my instance 
* sure enough, there were multiple bugs that needed to handle when deploying my Item catalog to my instance (although, it worked fine on my local development environment). The login using Google OAuth is but one example that needed few updates for it to work.
* the instance also needed every path to be hardcoded (these are mainly the database path and client_secert.json file)
* I needed to make sure that every connection when opened is closed before returning any responses.
* finally, I was able to knock out all bugs and had the Item Catalog up and running on this URL: [My Favorite Manga](http://52.57.211.236.xip.io)
* Of course, needless to say, that I did install all dependencies beforehand (those are detailed in the following section)
## (req.iv) list of any third-party resources you made use of to complete this project.
---
Below is a list of 3rd party resources I used in this project:

* python3-pip
* Flask
* SQLite
* httplib2
* requests
* oauth2client
* random
* string
* sqlalchemy
* jinja2
* json
* functools

## (req.iv) list of any third-party resources you made use of to complete this project.
---
Below is a list of 3rd party resources I used in this project:
* [Getting to Know Linux File Permissions | Linux.com | The source for Linux information](https://www.linux.com/learn/getting-know-linux-file-permissions)
* [Flask Hello World App with Apache WSGI on Ubuntu 14 - 2018](https://www.bogotobogo.com/python/Flask/Python_Flask_HelloWorld_App_with_Apache_WSGI_Ubuntu14.php)
* [Python: How to project folder (cant find .json file) [Errno 2] - Programming - Linus Tech Tips](https://linustechtips.com/main/topic/898357-python-how-to-project-folder-cant-find-json-file-errno-2/)
* [Error Messages — SQLAlchemy 1.3 Documentation](https://docs.sqlalchemy.org/en/latest/errors.html#error-f405)
* [SQLAlchemy in Flask — Flask 0.10.1 documentation](https://flask-doc.readthedocs.io/en/latest/patterns/sqlalchemy.html)
