## Preparing software packages

Following the manual installation suggestions, rbenv is used to install Ruby. Whenever possible, I use all four cores to speed up compiling (hence the **-j 4** flags). These tasks have to be done as the openproject user, hence the **su -- openproject** command.  

```
sudo su -- openproject -login
git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.profile
echo 'eval "$(rbenv init -)"' >> ~/.profile
source ~/.profile
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
MAKE_OPTS="-j 4" rbenv install 2.7.2
rbenv rehash
rbenv global 2.7.2

```
This will take 15 minutes. 

Earlier version of the protocol required an older (2.6) version, now 2.7.2 compiles just fine. 

Check version with
```
ruby --version
```

Following the manual installation suggestions, nodenv is used to install Ruby. Whenever possible, I use all four cores to speed up compiling (watch out for the **-j 4** flags).

```
git clone https://github.com/OiNutter/nodenv.git ~/.nodenv
echo 'export PATH="$HOME/.nodenv/bin:$PATH"' >> ~/.profile
echo 'eval "$(nodenv init -)"' >> ~/.profile
source ~/.profile
git clone git://github.com/OiNutter/node-build.git ~/.nodenv/plugins/node-build
MAKE_OPTS="-j 4" nodenv install 13.7.0
nodenv rehash
nodenv global 13.7.0
```

This will take 1 minute.  

## Compile and install OpenProject

Careful - the manual installation I linked to above still uses stable/9, the current release is stable/11 (as of Feb 2021). So, using release stable/11 here. Earlier issues with bcrypt and other gems appear to have resolved themselves. 

```
cd ~
git clone https://github.com/opf/openproject.git --branch stable/11--depth 1
cd openproject
gem update --system 
gem install bundler
bundle install --deployment --without mysql2 sqlite development test therubyracer docker 
npm install
npm audit fix
```

**gem update --system** will take about 8 minutes.

**bundle install** will take about 5 minutes.

**npm install** will take  15 minutes.


## Prepare config files

### Database:
```
cp config/database.yml.example config/database.yml
nano config/database.yml
```
```
production:
  adapter: postgresql
  encoding: unicode
  database: openproject
  pool: 20
  username: openproject
  password: openproject
```
  
### Email & memcache:
 
Create an app password for gmail, and include it in the config file. Make sure to keep the .yml layout intact. 
```
   cp config/configuration.yml.example config/configuration.yml
 nano config/configuration.yml
```
```
production:                          
  smtp_address: smtp.gmail.com
  smtp_port: 587
  smtp_domain: smtp.gmail.com
  smtp_user_name: **@gmail.com**
  smtp_password: **enter gsuite app password here**
  smtp_enable_starttls_auto: true
  smtp_authentication: plain
  
** Add at the end of the file:**
 
 rails_cache_store: :memcache
```


  
## Setup OpenProject
Set secret key and store in environmental variable SECRET_KEY_BASE. 

```
su -- openproject -login
cd ~/openproject
echo "export SECRET_KEY_BASE=$(./bin/rake secret)" >> ~/.profile
source ~/.profile
RAILS_ENV="production" ./bin/rake db:create
RAILS_ENV="production" ./bin/rake db:migrate
RAILS_ENV="production" ./bin/rake db:seed
RAILS_ENV="production" ./bin/rake assets:precompile
exit
```
Last one is the slow one - expect to wait for 15 minutes. 


## Install Apache & Passenger

```
sudo apt-get install -y apache2 libcurl4-gnutls-dev apache2-dev libapr1-dev libaprutil1-dev
sudo chmod o+x "/home/openproject"
sudo apt-get install -y dirmngr gnupg apt-transport-https ca-certificates curl
curl https://oss-binaries.phusionpassenger.com/auto-software-signing-gpg-key.txt | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/phusion.gpg >/dev/null
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bullseye main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
sudo apt-get install -y libapache2-mod-passenger
```
This will take about 5 minutes.

Then, still as root:
```
sudo a2enmod passenger
sudo systemctl restart apache2
```

Create the /etc/apache2/sites-available/openproject.conf file:
```
<VirtualHost *:80>
   ServerName yourdomain.com
   # !!! Be sure to point DocumentRoot to 'public'!
   DocumentRoot /home/openproject/openproject/public
   <Directory /home/openproject/openproject/public>
      # This relaxes Apache security settings.
      AllowOverride all
      # MultiViews must be turned off.
      Options -MultiViews
      # Uncomment this if you're on Apache >= 2.4:
      Require all granted
   </Directory>

   # Request browser to cache assets
   <Location /assets/>
     ExpiresActive On ExpiresDefault "access plus 1 year"
   </Location>

</VirtualHost>
```
 
Reboot or restart the apache service again: 

```
sudo a2ensite openproject
sudo systemctl restart apache2
```


And now: OpenProject should be accessible on **raspberry_pi-IP**:80. The standard login is admin // admin. If this does not work, see **Troubleshooting** below. 

## Email notifications

The template configuration above works, if an 'app specific password' is created in the corresponding google account, and the POP/IMAP option is activated for the same account. See details here: https://support.google.com/accounts/answer/185833?hl=en

## Congratulations!


For email notification to work well, background jobs have to be enabled. This is untested. 

Switch back to the user openproject, and edit crontab:

```
sudo su -- openproject -login
crontab -e **select the editor of choice if required**
```

Insert the following line at the end of the file, make sure to use the correct Ruby version:

```
*/1 * * * * cd /home/openproject/openproject; . ~/.profile; RAILS_ENV="production" ./bin/rake jobs:workoff
```
In Rails 6 or later, "config.cache_classes" must be set to false:

```
nano /home/openproject/openproject/environments/production.rb
***Find the line config.cache_classes=true and change it to false***
```

Save & reboot. Enjoy. 

## Final Thoughts

Given the limited experience of how well this works, and if it will survive future development and updates, think carefully about whether you want to use this installation for anything other than a hobby, home or testing environment. OpenProject might feel inspired to make ARM a supported architecture in the future (clearly, it shouldn't be too much of a problem!). 

And obviously, if you plan to make this installation available on any sort of network, change the various passwords and user accounts, and shore up security on Raspian!
