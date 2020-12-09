# **How To Install WordPress On Ubuntu v16.04 Server**
_Gryphon Strother_ - _01/12/2020_

![WordPress Graphic](http://web-13.scwebsrv.com/public_html/web105/fikret-tozak-Zk--Ydz2IAs-unsplash.jpg)

Today I will show you how to fully install WordPress on your server! I will assume that you have configured your server with a **static ip address,** 
that it has **SSH open,** and that it is **connected to the Internet.**


## Before we go any further, there are **two things you should know:**  
 When you see [`square brackets with text inside`] this indicates a **user defined input.** 
>Text appearing like this indicates a command you will need to run

With that out of the way, lets get on with our installations.

From now on we will be using **Terminal** (OSX) or **Command Prompt** (Windows) for console commands.

## First, we will need to **SSH to our Virtual Machine.**

Enter this command line to connect to your server:
>[`server username`]@[`server ip address`] -p [`server SSH port`]

**Example:** _myserver@123.456.78.90 -p 443_

If successful, you will be required to enter your Virtual Machine's password. Enter it you will be greeted with this familiar welcome screen:

![SSH Welcome Screen Image](http://web-13.scwebsrv.com/public_html/web105/wpinstallstart.png)

## Let's begin the install process by installing **Apache** in your Linux VM console.

Install Apache, then restart your network by running these two commands in order:

>sudo apt-get install apache2
>sudo service networking restart  

Navigate to [`your ip address`] in a browser and **if you are greeted with the page shown below then you have successfully installed Apache and may move on.**

![Apache2 Default Page](http://web-13.scwebsrv.com/public_html/web105/ubuntu%20default%20page.PNG)

## Next, let's configure our **domain**.

First, determine what you would like to be your root web directory.

By default in Apache this is **/var/www/html** so that is what we will be using for this tutorial. 

If you wish to create a new folder for your web directory, navigate to where you would like to create it and do so with:

>sudo mkdir [`folder name`]

Second, we will edit the Apache hosts file and add an entry for our domain:

>sudo nano /etc/hosts

Add an entry for:

>127.0.0.1 [`yourdomain.com`] 

below the "localhost" entry like the **highlighted text** in the image below

![Hosts file entries](http://web-13.scwebsrv.com/public_html/web105/hosts%20file%20entry.PNG)

**To save and exit this and any file you edit using nano, press [`CTRL O`] to save, [`ENTER`] to confirm and [`CTRL X`] to exit back to the command line.**

Third, we will copy the default Apache configuration file to a new custom one we will edit by first navigating to the directory with:

>cd /etc/apache2/sites-available

Then we copy the default file to a new file:

>sudo cp 000-default.conf [`yourdomain.com.conf`]

Once copied, let's make some changes:

>sudo nano [`yourdomain.com`].conf

**We will need to:**
- **Remove the "#" beside "ServerName"** and add a [`yourdomain.com`] entry beside it.
- Change the **DocumentRoot** path to lead to your **web root directory** 

![Apache conf file changes](http://web-13.scwebsrv.com/public_html/web105/apache%20conf%20file%20changes.PNG)

Finally, to finish configuring our domain, we will need to disable the default configuration and enable our newly created one:

>sudo a2dissite 000-default.conf 
sudo a2ensite [`yourdomain.com`].conf 

Then restart apache and you're done!

>sudo service apache2 restart 

![Apache configured](http://web-13.scwebsrv.com/public_html/web105/apache%20success.PNG)

**If you keep hitting the Default Apache Page when visiting your domain in a browser, navigate to your web root and ensure there isn't a index.html file there**
 _(If there is, this is the Default Apache Page that is auto loading when you visit your site. Remove or move this file elswhere to fix this.)_

## Now it's time to begin installing **PHP** and **MySQL**.
#### Let's start with **MySQL**.
Run the install command:

>sudo apt-get install mysql-server

You will be asked for your server password, and will be again asked again every time you are installing something on your server. 

Press **y** when you are prompted to continue the installation.

Shortly after, the console will turn pink and you will be pormpted to enter a password for the MySQL "root" user:

![MySQL root user password screen](http://web-13.scwebsrv.com/public_html/web105/mysqlpass.png)

**Create a password and record it as it will be needed later on.** After the install finishes, let's make sure its working:
>sudo service mysql status

Check that it is **Active: active (running)** and then restart the service to complete the install:

>sudo service mysql restart

#### Let's do **PHP** next:

The first thing we will need to do is add a respository (repo for short) that contains PHP:

>sudo add-apt-repository ppa:ondrej/php 


During install it will prompt you, press **Enter** to finish the install.

Next, we need to run the update command to make sure that our system is utilizing the new repo:

>sudo apt-get update

Once the update is finished, it's time to install the base PHP package:

>sudo apt-get install php7.2

You will be prompted asking if it is okay to use disk space, press **y** to continue with the install.

Finally, it's time to install the requried PHP mods and plugins needed for WordPress to function on the server:

>sudo apt-get install libapache2-mod-php7.2
sudo apt-get install php7.2-mysql
sudo apt-get install php7.2-mbstring

Let's also run a status command to check that MySQL is still working, and then restart it to make sure all of our changes are saved. 

>sudo service mysql status
sudo service mysql restart

And we're done installing!

## Time to begin the **WordPress** install!

The process of installing WordPress is the longest and requires multiple steps. The first one we will take is downloading the [WordPress](http://www.wordpress.org/latest.zip) files directly to our web directory.

I lied, restart Apache first to be extra safe:

>sudo service apache2 restart

**Now** lets download the latest WordPress files to our server. 

First, navigate to your web root and create a folder for WordPress to live in:

>sudo mkdir [`wordpress folder name`]

Next, we will use the curl command to download the latest WordPress files directly to our server. This can be done with a **FTP** (File Transfer Protocol) client but **running the curl command
 will nearly always be _much_ faster** as we will be **downloading** the files directly from the website to our server rather than **uploading** to our server from our local machine after we download them from the WordPress website.

In order to upload or download files to this directory, we first must allow read and writes: _**(run this command right below the new folder you just made as shown in the figure below)**_ 

>sudo chown -R [`linux username`] [`foldername`]

![Where to chown](http://web-13.scwebsrv.com/public_html/web105/where%20to%20chown.PNG)


To curl, navigate inside your new folder and run:

>curl -O `https://wordpress.org/latest.zip`

This will take some time (depending on your connection speed.) When finished, you will need to run this command to install **UnZip**, a useful tool we will use to unzip this archive:

>sudo apt-get install unzip

After it installs, run this command to unzip the WordPress files you just downloaded:

>unzip latest.zip

WordPress should now be unzipped and ready to be configured. Restart Apache to be safe:

>sudo service apache2 restart

**Now navigate to your domain in a browser, click on _the folder you created for WordPress to live in_ and then the _"wordpress"_ folder inside of it.** 

You should be greeted with this screen if everything was done correctly:

![WP first screen](http://web-13.scwebsrv.com/public_html/web105/wp%20first%20screen.PNG)

#### The next step of the **WordPress** installation is done in **MySQL**.

First, check to make sure MySQL hasn't died on us:

>sudo service mysql status

If its alive, and **active (running)** then let's get into the MySQL CMD:

>sudo mysql -u root -p

You will be required to enter **the MySQL password you created when installing MySQL** in order to get in.

Once in, run this to create the database:

>CREATE DATABASE [`name your wordpress database here`] [` ; `]

**The [` ; `] signifies a semicolon press after pressing [`ENTER`] and running your command in MySQL.** You will need to **press [`ENTER`] again after typing the semicolon** as if you are running a command.

Then run this:

>CREATE USER '[`choose a username`]'@'localhost' IDENTIFIED BY '[`choose a password`]' [` ; `]

**Example:** _CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'gs'_

After, run this to grant all permissions:

>GRANT ALL PRIVILEGES ON [`database name`].* TO '[`database username`]'@'localhost' [` ; `]

**Example:** _GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost'_

Next, let's check to make sure our database was created:

>SHOW DATABASES [` ; `]

You should see the newly created database listed. 

Now let's run one more command to finish up our database work:

>FLUSH PRIVILEGES [` ; `]

And this to exit MySQL:

>exit

Before proceeding, it's good practice to restart Apache and MySQL:

>sudo service mysql restart
sudo service apache2 restart

#### Back to the browser to enter in our **database details.**

Return to the **Welcome to WordPress** page we landed on before configuring the database in MySQL. 

Click the blue **"Let's go!"** button to proceed.

On the next page, enter the information for the database you just created in **MySQL**:

![WP database info](http://web-13.scwebsrv.com/public_html/web105/wp%20database%20info.PNG)

( You can leave the **Database Host** and **Table Prefix** at their default settings as seen in the image )

After **double-checking your entries** click **"Submit"** to proceed.

You will be taken to this page:

![WP config info](http://web-13.scwebsrv.com/public_html/web105/wp%20config%20info.PNG)

The top texts notes that WordPress was **Unable to write to wp-config.php file**.

Navigate to the **"wordpress"** folder located inside the folder you made for WordPress to live in.

**Example Location:** _/var/www/html/wordpresstutorial/wordpress_

Run the [` ls `] command to make sure you are in the right directory.

![WP LS for config file](http://web-13.scwebsrv.com/public_html/web105/wp%20making%20config%20file.PNG)

Next we will create a **"wp-config.php"** file:

>sudo nano wp-config.php

**Copy all the text in the grey text box by clicking inside the box, pressing [` CTRL A `] or [` COMMAND A `] and then [` CTRL C `]** 

![WP Config Copy Box](http://web-13.scwebsrv.com/public_html/web105/wp%20config%20info%20-%20Copy.PNG)

**Paste all the text inside the _wp-config.php_ file and save it.**

_On Windows, **Right Click** to paste inside a **Command Prompt** window._

## You've made it to the **final step!**

Click **Run The Installation** after you save the config file, and it should take you to this page:

![WP Final step](http://web-13.scwebsrv.com/public_html/web105/wp%20final%20step.PNG)

From here, add a **Site Title, Username, Password, and Email** and click **"Install WordPress" to finish the install!

## You did it! **Enjoy your new WordPress website!**

Take a moment to pat yourself on the back :)

![Final Image](http://web-13.scwebsrv.com/public_html/web105/stephen-phillips-hostreviews-co-uk-sSPzmL7fpWc-unsplash.jpg)



























