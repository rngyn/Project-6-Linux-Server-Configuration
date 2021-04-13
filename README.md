# Linux Server Configuration Project
The Linux Server Configuration Project shows how to configure and deploy a Linux
server to host projects that are accessible to the public via the internet. This
particular project uses a server provided by [Amazon Web Services](https://aws.amazon.com/lightsail/)
through Lightsail. It will host a previous project, the Lens Catalog with full
features.

* Public IP Address: [54.191.158.8](http://54.191.158.8/) (does not support OAuth 2.0)
* Domain Name URL: [http://ec2-54-191-158-8.us-west-2.compute.amazonaws.com](http://ec2-54-191-158-8.us-west-2.compute.amazonaws.com)
(working OAuth 2.0)
* Port: 2200

The following steps were done to get this up and running...

## Get your server.
1. [Amazon Lightsail](https://lightsail.aws.amazon.com) was used as the virtual
server for this project. Follow the link to log in, or to sign up for a new account.
2. Create an instance with a **Linux/Unix** platform, then clicking *OS Only* under
blueprint and choosing **Ubuntu**.
3. The cheapest plan at $5 USD/month is sufficient.
4. Give the instance a name. For this project, the name **linux-server-configuration**
was given.
5. After a moment, the instance will start running. You can click it to see more settings,
and more importantly, the public IP, which is **54.191.158.8** for this project.
6. Click the **Connect using SSH** button to begin setting up the server.

## Secure Your Server
### Update Files
* Once the browser terminal is up and running, update all currently installed
packages by running:
```
sudo apt-get update
sudo apt-get upgrade
```

### Change SSH Port
* Now we need to change the SSH port from the default **22** to **2200**. This can
be done by running:
```
sudo nano /etc/ssh/sshd_config
```
and changing `Port 22` to `Port 2200`. `PermitRootLogin no` should be set as well.
Hit `ctrl + X` to exit, then `y` and `enter` to confirm changes.

* Restart SSH with:
```
sudo service ssh restart
```

### Configure UFW
* Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/tcp
sudo ufw enable
```

### Change Lightsail Firewall Settings
* From here, you can try configuring Lightsail's firewall settings under the **Networking**
tab by adding a *Custom TCP 2200* setting, but I found it to not work very well for me
so alternatively, I installed **PuTTY** to continue working on the project after the SSH port change.

[SSH Configuration Source](https://serversforhackers.com/c/configuring-sshd-on-the-server)

## Installing PuTTY
### Download and Install
* Download the latest version of [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

### Get Private Key
* On Lightsail, on the **Account** page, download the default private key for the instance
under the **SSH keys** tab. Save the `.pem` file wherever easily found.

### Configure PuTTYgen With Private Key
3. Run the application `PuTTYgen` from your computer.
4. Click `Load` and browse to the `.pem` file and open it. Keep in mind the
browse window defaults to only showing `.ppk` files, so be sure to display all file
types.
5. Click `OK` once the key has been successfully imported.
6. Click `Save private key` and confirm to save it without a passphrase.
7. Name the `.ppk` file and save it somewhere.
8. Close `PuTTYgen`.

### Configure PuTTY and Log In
9. Open `PuTTY`.
10. Take note of the Lightsail Public IP address of the instance (54.191.158.8 in this case)
and copy it into the `Host Name (or IP address)` field.
11. Remember to change the `Port` to **2200**.
12. In the `Category` window, expand `SSH` and click `Auth`.
13. Click `Browse...` and find the `.ppk` file you created, and click `Open`.
14. Click `Open` again and `Yes` to trust the connection.
15. A terminal window will open, log in as `ubuntu` and you'll be logged back into your
instance.

[PuTTY Setup Source](https://lightsail.aws.amazon.com/ls/docs/how-to/article/lightsail-how-to-set-up-putty-to-connect-using-ssh)

## Give `grader` Access.
### Creating a New User and Giving `sudo` Access
1. Create a new `grader` user:
```
sudo adduser grader
```
2. Give the `grader` user any password, and fill out additional information if desired.
3. Give this new user `sudo` access by editing the following file:
```
sudo nano /etc/sudoers.d/grader
```
and adding the line `grader ALL=(ALL) NOPASSWD:ALL`. Save, and exit.

### Creating an SSH Key Pair
1. Run a terminal window on your local computer with the following line:
```
ssh-keygen
```
2. Enter a file name to save. I just left it as default.
3. Leave the passphrase empty.
4. In the same terminal window, run:
```
cat .ssh/id_rsa.pub
```
or whatever your file name and path is for the file you just created.
5. Copy all the contents of that file.
6. On your instance terminal window, run the following commands to create the
appropriate files:
```
sudo mkdir .ssh
sudo touch .ssh/authorized_keys
sudo nano .ssh/authorized_keys
```
7. Paste the long string into the file. Save, and exit.
8. Set the permissions with the following commands:
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```

## Prepare and Deploy Your Project
### Change the Local Time Zone to UTC
* This can be done by running:
```
sudo dpkg-reconfigure tzdata
```
select `None of the above` and then `UTC`.

### Install Apache
* Run:
```
sudo apt-get install apache2
```

### Folder Setup
2. Now we need to prepare the folder structure with the following series of commands:
```
cd /var/www
sudo mkdir CatalogApp
cd CatalogApp
sudo mkdir CatalogApp
cd CatalogApp
sudo mkdir static templates
```
The file structure within `/var/www` should look like:
```
|----CatalogApp
|---------CatalogApp
|--------------static
|--------------templates
```
3. Create an `__init__.py` file within the `/var/www/CatalogApp/CatalogApp` directory:
```
sudo touch __init__.py
```

### Installing Dependencies
* Run the following commands to install required packages:
```
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install python-dev python-pip
sudo apt-get install python-flask python-sqlalchemy
sudo pip install oauth2client httplib2 requests psycopg2
```

### Configuring the Host
1. We need to populate the `.conf` file with correct configurations. Start by running:
```
sudo nano /etc/apache2/sites-available/CatalogApp.conf
```
then populating the file with:
```
<VirtualHost *:80>
		ServerName http://54.191.158.8/
		ServerAdmin admin@54.191.158.8
		WSGIScriptAlias / /var/www/CatalogApp/catalogapp.wsgi
		<Directory /var/www/CatalogApp/CatalogApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/CatalogApp/CatalogApp/static
		<Directory /var/www/CatalogApp/CatalogApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Save, and exit.
2. We need to disable the default `.conf` file and enable the new one:
```
sudo a2dissite 000-default.conf
sudo a2ensite CatalogApp.conf
```
3. Now, create and edit the `.wsgi` file:
```
cd /var/www/CatalogApp
sudo nano catalogapp.wsgi
```
And add the following code:

        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/CatalogApp/")

        from CatalogApp import app as application
        application.secret_key = 'super_secret_key'

      Save, and exit.
4. Restart Apache:
```
sudo service apache2 restart
```

[Timezone Change Setup Source](https://askubuntu.com/questions/323131/setting-timezone-from-terminal), [Apache Setup Source](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

### Installing and Configuring PostgreSQL
1. Install PostgreSQL using:
```
sudo apt-get install postgresql
```
2. We can now log in as default user:
```
sudo su postgres
```
3. Enter the shell with the `psql` command and execute the following to create the database and `catalog` user:
```
CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog;
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
4. Exit the shell with:
```
\q
```

### Installing Git
* Git can be installed by running:
```
sudo apt-get install git
```

[PostgreSQL Setup Source](https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e)

## Deploy the Item Catalog Project
1. We need to clone the project from GitHub. Make sure you're in the `/var/www/CatalogApp`
directory by running:
```
cd /var/www/CatalogApp
```
2. Now clone the project:
```
sudo git clone https://github.com/rngyn/Lens-Catalog.git
```
3. And make sure to rename the `Lens-Catalog` directory to `CatalogApp` to nest it in
the correct folder structure:
```
sudo mv /var/www/CatalogApp/Lens-Catalog /var/www/CatalogApp/CatalogApp
```
4. Confirm the project files are in the right place by running the commands:
```
cd /var/www/CatalogApp/CatalogApp/
ls
```

### Configuring the Project Files
1. The database engine needs to be rerouted for the project to function correctly.
This is done by changing
```
engine = create_engine('sqlite:///lenscatalog.db')
```
to
```
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```
in each of these files: `database_setup.py`, `lensinit.py`, and `project.py`.
2. Rename the `project.py` file to `__init__.py`
```
sudo mv project.py __init__.py
```

### Configuring for OAuth 2.0
1. To get OAuth 2.0 working correctly for the project, the `__init__.py` needs to
be edited so it can successfully point to the right `.json` files. This is done by
routing it to its absolute path. For Google, change:
```
client_secrets.json
```
to
```
/var/www/CatalogApp/CatalogApp/client_secrets.json
```
in the following spots in the code:
```
CLIENT_ID = ...
@app.route('/gconnect'... oauth_flow = ...
```
2. For Facebook:
```
fb_client_secrets.json
```
change to
```
/var/www/CatalogApp/CatalogApp/fb_client_secrets.json
```
in the following spots in the code:
```
@app.route('/fbconnect'... app_id = ...
@app.route('/fbconnect'... app_secret = ...
```
3. Next, the you must add the domain name URL to the OAuth 2.0 client so authentication
can function properly. For Google, log into your [Google APIs](https://console.developers.google.com/apis)
account, click the appropriate project, go into the **Credentials** then add the following
to the **Authorized JavaScript origins**:
```
http://54.191.158.8
http://ec2-54-191-158-8.us-west-2.compute.amazonaws.com
```
Then add the following to the **Authorized redirect URIs**:
```
http://ec2-54-191-158-8.us-west-2.compute.amazonaws.com/login
http://ec2-54-191-158-8.us-west-2.compute.amazonaws.com/gconnect
```
4. A new `client_secrets.json` must now be transferred to the project.
Do this by first downloading the `.json` file from Google APIs, opening the file and copying
the contents.
5. Back in the terminal window of your instance, run:
```
sudo nano /var/www/CatalogApp/CatalogApp/client_secrets.json
```
Delete the original file contents, then pasting in the new `.json` information.
6. For Facebook, log into the [Facebook Developers](https://developers.facebook.com)
webpage, go into the **Basic Settings** of the app, and pasting:
```
http://ec2-54-191-158-8.us-west-2.compute.amazonaws.com
```
into the **Site URL** field. No further configuration is required.

7. The application should now be able to load and function properly from both

	[http://54.191.158.8/](http://54.191.158.8/)

	and

	[http://ec2-54-191-158-8.us-west-2.compute.amazonaws.com](http://ec2-54-191-158-8.us-west-2.compute.amazonaws.com)

	However, due to Google's recent update of not allowing IP for redirects, only accessing
	the application through the domain name URL has the login function working with
	both Google and Facebook sign in.

[Database Engine Reroute Source](http://docs.sqlalchemy.org/en/latest/core/engines.html),
[Configuring for OAuth 2.0 Source](https://discussions.udacity.com/t/google-oauth-api-and-authorized-redirect-uris-with-ip-adr-not-allowed/515313/3)
