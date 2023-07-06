# Troubleshooting

2020/02/25
2020/02/26
2020/05/05

### I made it through the process, the openproject login page shows up, but I cannot use the admin // admin login. 

You might have missed an error that triggered during the **RAILS_ENV="production" ./bin/rake db:seed** step. When the WorkPackages section is executed, a **getaddrinfo** error is triggered, and **rake aborted** is displayed. 
This can be easily fixed by executing the following commands as a superuser (e.g. from the pi account), to make sure that /etc/hosts and /etc/resolv.conf can be read by all users. This doesn't seem to reliably fix the problem - removed from the above instructions. 2020/02/26 

```
sudo chmod o+r /etc/resolv.conf
sudo chmod o+r /etc/hosts
```
### I still cannot login with admin // admin

You can manually change the password for the admin account by opening an IRB terminal. This is generally useful if a user forgot their password, and outlined originally [here](https://community.openproject.com/topics/6584).

As the openproject user, open an IRB shell:

```
cd ~/openproject
RAILS_ENV=production bundle exec rails c
```
Once the console is open, type out the following:
```
admin = User.find_by(login: 'admin')
admin.password = 'RaspberryPi' # Must confirm to the password rules you defined!
admin.password_confirmation = 'RaspberryPi'

admin.save! # Watch the output for errors
```
The final command should return 'true'. Use the new password to login.

### Error page when accessing the login page & error message the error log:
"Could not spawn process for application /home/openproject/openproject: A timeout occurred while starting a preloader process."

Discovered, solved & reported by Matteo Nespoli (thanks!): Add "PassengerStartTimeout 200" line to passenger.conf, as outlined here: https://community.bitnami.com/t/openproject-8-3-2-0-stack-not-working-after-fresh-install/67444/12

### Test emails work, but notifications don't arrive

Background jobs are probably not activated, follow these steps: 

https://docs.openproject.org/installation-and-operations/installation/manual/

### More questions?

Email or ask here: https://community.openproject.com/topics/6873

