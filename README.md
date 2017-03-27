Note: as of 3-27-2017 the latest version Ghost-alpha.17 is broken
# Install Ghost 1.0+ on DigitalOcean 512 MB droplet 
<blockquote style="font-family: monospace;">
Build a clonable Ghost server platform on a DigitalOcean 512 MB droplet with Ubuntu, MariaDB, Nginx, all the apps needed to run Ghost, unattended server upgrades, and daily automatic snapshots. Make a snapshot to clone present and future Ghost sites. Then install SSL encrypted Ghost and connect it to KeyCDN, MailChimp, Disqus, Google Analytics, and Google Custom Search. The whole process takes about a half-hour from zero to speedy, secure Ghost blog.
</blockquote>
<pre>
<h2>1. Spin up a DigitalOcean 512 MB droplet</h2>
Open generic-ghost-commands.html in a text editor and perform a find and replace on these twelve values. 
Save and reopen in a browser. All commands will be populated with your data for easy copy/paste. 
(You may need to modify tabs and spacing to make the page format correctly for your browser.)

	Domain: 			yourweb.com
	Domain base: 			yourweb
	Server IP (floating): 		123.123.123.123
	UNIX root password:   		unix-rootpassword
	DB root password: 		maria-rootpassword
	Non-root unix sudo user: 	non-rootunixuser
	Non-root unix password: 	non-rootunixpassword
	Non-root MariaDB user: 		non-rootmariauser
	Non-root MariaDB password: 	non-rootmariapassword
	DO API key: 			qwertyuioplkjhgfdsazxcvbnmpoiuytrewq1234567890987654321
	Droplet ID: 			12345678
	Key CDN account:		123b.kxcdn.com
	
<h5>LOG IN</h5>
	YourComputer :~ youruser$ ssh root@123.123.123.123
	Changing password for root.(current)
	UNIX password:Enter new
	UNIX password:Retype new UNIX password:
<h5>CREATE NEW UNIX USER, then LOG OUT</h5>
	root@yourweb:~# adduser non-rootunixuser
	root@yourweb:~# gpasswd -a non-rootunixuser sudo
	root@yourweb:~# usermod -a -G www-data non-rootunixuser
	root@yourweb:~# exit
<h5>LOGIN AS NON-ROOT UNIX USER, add DigitalOcean AGENT, run UPDATES</h5>
	YourComputer :~ youruser$ ssh non-rootunixuser@123.123.123.123
	non-rootunixuser@yourweb:~# curl -sSL https://agent.digitalocean.com/install.sh | sh
	non-rootunixuser@yourweb:~$ sudo apt-get upgrade
	non-rootunixuser@yourweb:~$ sudo apt-get dist-upgrade
	non-rootunixuser@yourweb:~$ sudo apt-get update
	non-rootunixuser@yourweb:~$ sudo apt-get autoremove
	non-rootunixuser@yourweb:~$ sudo apt-get autoclean
<h5>MAKE SWAP FILE</h5>
	non-rootunixuser@yourweb:~$ sudo fallocate -l 1G /swapfile
	non-rootunixuser@yourweb:~$ sudo chmod 600 /swapfile
	non-rootunixuser@yourweb:~$ ls -lh /swapfile
	non-rootunixuser@yourweb:~$ sudo mkswap /swapfile
	non-rootunixuser@yourweb:~$ sudo swapon /swapfile
	non-rootunixuser@yourweb:~$ echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
	non-rootunixuser@yourweb:~$ sudo sysctl vm.swappiness=10
	non-rootunixuser@yourweb:~$ sudo sysctl vm.vfs_cache_pressure=50
	non-rootunixuser@yourweb:~$ sudo nano /etc/sysctl.conf
		At the bottom add:
		vm.swappiness=10
		vm.vfs_cache_pressure=50

<hr>
<h2>2. Build Ghost Server Platform:</h2>
<h3>MARIA-DB</h3>
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ sudo apt-get install software-properties-common
non-rootunixuser@yourweb:~$ sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
non-rootunixuser@yourweb:~$ sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://mirrors.syringanetworks.net/mariadb/repo/10.1/ubuntu xenial main'
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ sudo apt install mariadb-server
non-rootunixuser@yourweb:~$ sudo mysql_secure_installation

<h5>ADD NEW MARIA USER</h5>
non-rootunixuser@yourweb:~$ sudo mysql -u root -pmaria-rootpassword
	CREATE USER 'non-rootmariauser'@'localhost' IDENTIFIED BY 'non-rootmariapassword';
	GRANT ALL PRIVILEGES ON * . * TO 'non-rootmariauser'@'localhost' with grant option;
	GRANT PROXY ON ''@'' TO 'non-rootmariauser'@'localhost' with grant option;
	FLUSH PRIVILEGES;
	exit

<h3>NGINX</h3>
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$  sudo apt-get install nginx
non-rootunixuser@yourweb:~$ sudo ufw allow 'Nginx HTTP' && sudo ufw allow 'Nginx HTTPS' && sudo ufw allow http && sudo ufw allow https && sudo ufw allow ssh && sudo ufw disable && sudo ufw enable

<h3>GHOST APPS</h3>
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
non-rootunixuser@yourweb:~$ sudo bash nodesource_setup.sh
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ sudo apt-get install nodejs
non-rootunixuser@yourweb:~$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
non-rootunixuser@yourweb:~$ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ sudo apt-get install yarn
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ sudo yarn global add ghost-cli
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ sudo npm i -g ghost-cli
non-rootunixuser@yourweb:~$ sudo apt-get update

<h3>UNATTENDED UPGRADES</h3>
To counter the biggest threat to software packages, they should be updated on a regular basis. 
Vulnerabilities are discovered on a daily basis, which also requires we monitor daily. 
Software patching takes time, especially when testing and reboots are needed. 
Fortunately, systems running Debian and Ubuntu can use unattended-upgrades
to achieve automated patch management for security updates.

non-rootunixuser@yourweb:~$ sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
	uncomment --> 	"${distro_id}:${distro_codename}-updates";
	modify --> 	Unattended-Upgrade::Mail "admin@yoursite.com";
	modify --> 	Unattended-Upgrade::Remove-Unused-Dependencies "true";
	modify --> 	Unattended-Upgrade::Automatic-Reboot "true";
	modify --> 	Unattended-Upgrade::Automatic-Reboot-Time "02:00";	

non-rootunixuser@yourweb:~$ sudo nano /etc/apt/apt.conf.d/10periodic
	modify --> 	APT::Periodic::AutocleanInterval "7";
	add line -->	APT::Periodic::Unattended-Upgrade "1";

<h3>AUTO SNAPSHOT</h3>
non-rootunixuser@yourweb:~$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
non-rootunixuser@yourweb:~$ sudo \curl -sSL https://get.rvm.io | bash -s stable --rails
non-rootunixuser@yourweb:~$ source /home/non-rootunixuser/.rvm/scripts/rvm
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ sudo wget https://assets.merqlove.ru.s3.amazonaws.com/do_snapshot/do_snapshot.tgz --no-check-certificate
non-rootunixuser@yourweb:~$ sudo tar -xzf do_snapshot.tgz
non-rootunixuser@yourweb:~$ sudo cp -r do_snapshot /usr/local/
non-rootunixuser@yourweb:~$ sudo ln -s /usr/local/do_snapshot/bin/do_snapshot /usr/local/bin/do_snapshot
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ gem install do_snapshot
non-rootunixuser@yourweb:~$ gem install rest-client
non-rootunixuser@yourweb:~$ rvm cron setup
non-rootunixuser@yourweb:~$ do_snapshot --digital-ocean-access-token qwertyuioplkjhgfdsazxcvbnmpoiuytrewq1234567890987654321 --only 12345678 -k 3 -c -v
non-rootunixuser@yourweb:~$  crontab -e
		03 00 * * * do_snapshot --digital-ocean-access-token qwertyuioplkjhgfdsazxcvbnmpoiuytrewq1234567890987654321 --only 12345678 -k 3 -c -v

<h3>PREPARE FOR GHOST INSTALL</h3>
non-rootunixuser@yourweb:~$ sudo apt-get upgrade
non-rootunixuser@yourweb:~$ sudo apt-get dist-upgrade
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:~$ sudo apt-get autoremove
non-rootunixuser@yourweb:~$ sudo apt-get autoclean
<strong>verify default Nginx page --> <a href="http://123.123.123.123">123.123.123.123</a></strong>
non-rootunixuser@yourweb:~$ sudo reboot

<h3>TAKE SNAPSHOT</h3>
At this point, you have built an easily clonable disk image for present and future Ghost installations. 
Make a snapshot now and name it something descriptive like "ghost-server-platform".

<hr>
<h2>3. Install Ghost, SSL, and CDN</h2>
Point your domain name A record to the server IP. Point a www cname to yourweb.com
<strong>verify the default Nginx page at each URL --> &nbsp; <a href="http://yourweb.com">yourweb.com</a> &nbsp; &nbsp; <a href="http://www.yourweb.com">www.yourweb.com</a> &nbsp; then, login and remove the default</strong>
YourComputer :~ youruser$ ssh non-rootunixuser@123.123.123.123
non-rootunixuser@yourweb:~$  sudo rm -rf /var/www/html/index.nginx-debian.html
non-rootunixuser@yourweb:~$ cd /var/www/html/
non-rootunixuser@yourweb:/var/www/html/$ sudo npm i -g ghost-cli@latest
non-rootunixuser@yourweb:~$ sudo apt-get update
non-rootunixuser@yourweb:/var/www/html/$ sudo ghost install
 ✔ Checking for latest Ghost version
 ✔ Running system checks
 ✔ Setting up install directory
 ✔ Downloading and installing Ghost v1.0.0-alpha.17
 ✔ Moving files
? Enter your blog URL: https://yourweb.com
? Enter your MySQL hostname: localhost
? Enter your MySQL username: non-rootmariauser
? Enter your MySQL password: *******************
? Enter your Ghost database name: ghost_production
 ✔ System Stack
? Do you want to set up your blog with SSL (using letsencrypt)? Yes
? Enter your email (used for ssl registration) info@yourweb.com
✔ Getting SSL certificate
✔ Generating encryption key (hold tight, this may take a while)
✔ Finishing setup
? Do you want to start Ghost? Yes

<h5>INCREASE FILE UPLOAD SIZE LIMIT</h5>
non-rootunixuser@yourweb:~$ sudo nano  /etc/nginx/sites-available/yourweb.com.conf
	add line in 443 server block: client_max_body_size 4M;
non-rootunixuser@yourweb:~$ sudo service nginx restart
<h3>ENABLE KeyCDN</h3>
Change the www cname DNS record to point to the keyCDN zone URL (e.g. yourweb-123b.kxcdn.com. )
Set up keyCDN zone and make a zonealias pointed to www.yourweb.com
Login to <a href="https://yourweb.com/ghost">https://yourweb.com/ghost</a> and add base href to the header code injection:
&lt;base href="https://www.yourweb.com/"&gt;
<h3>OTHER SERVICES ~ Child Theme</h3>
Upload CasperChild. Edit index.hbs and post.hbs to connect your MailChimp, Disqus, and Google Custom Search services. 
Place Google analytics code in the footer code injection field. 
non-rootunixuser@yourweb:~$ cd /var/www/html/
non-rootunixuser@yourweb:/var/www/html/$ sudo ghost restart

<hr><h2>Access your Ghost Blog at <a href="https://yourweb.com">https://yourweb.com</a></h2>

<hr>
<h3>UPDATE GHOST</h3>
non-rootunixuser@yourweb:~$ cd /var/www/html/
non-rootunixuser@yourweb:/var/www/html/$ sudo npm i -g ghost-cli@latest
non-rootunixuser@yourweb:/var/www/html/$ sudo apt-get update
non-rootunixuser@yourweb:/var/www/html/$ sudo ghost update
</pre>
</body>
</html>
