# Multi-Machine 
<div align="center">

![image](https://user-images.githubusercontent.com/104793540/185363187-90f460a6-1b1d-4a40-9dcc-ef8899e634b4.png)
  </div>
  
Step 1 - Vagrant file 
- In the vagrant file two machines will be created called app abd db
- the machines will be running in parrallel 

```ruby 
# Ruby
Vagrant.configure("2") do |config| 
# creating a virtual machine ubuntu 
    config.vm.define "app" do |app| # creates vm app
        app.vm.box = "ubuntu/bionic64" # setting up linux os 
        app.vm.network "private_network",  ip: "192.168.10.100" # network setup for nginx web server for app machine 
        app.vm.synced_folder ".", "/home/vagrant/app" # sync data from local host 
        app.vm.provision "shell", path: "provision.sh" # shell provisioner for script and location path 
    end

    config.vm.define "db" do |db| # creates vm db
        db.vm.box = "ubuntu/bionic64" # setting up linux os 
        db.vm.network "private_network",  ip: "192.168.10.150" # network setup for nginx web server for db machine
    end 
end 
```

Step 2 - Provisioning script with reverse proxy - App VM 
-  the provision command in the vargrant file [app.vm.provision "shell", path: "provision.sh]" finds the following .sh file which should be located in the same location of the root Vagrantfile for your project when  using path method

```provision.sh 
# updating and upgrading ubuntu 
 sudo apt-get update -y 
 sudo apt-get upgrade -y

 # setting up nginx 
 sudo apt-get install nginx -y
 sudo systemctl start nginx 
 sudo systemctl enable nginx 

 # setting up nodejs 
 sudo apt-get purge nodejs npm  
 curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
 sudo apt-get install -y nodejs  

 # setting up npm 
 sudo npm install pm2 -g
 sudo apt-get update -y
 sudo apt-get upgrade -y
 
 # automate reverse proxy 
 sudo cp -f app/rp_file/etc/nginx/sites-available/default
 sudo systemctl restarst nginx 
```

Automating script 

The reverse proxy can be automated by creating a new file with code we used earlier manually. Choose file location (i.e where app.js file is) 
- navigate to directory in app/app 
- `sudo nano rp_file`
- paste reverse proxy configuration here as shown below 

![image](https://user-images.githubusercontent.com/104793540/185062832-d0b140d3-e627-4ba2-876e-2f3fd00e6704.png)


```
server {
        listen 80;
        listen [::]:80;

        access_log /var/log/nginx/reverse-access.log;
        error_log /var/log/nginx/reverse-error.log;

        location / {
                    proxy_pass http://localhost:3000;
  }
}
```
![image](https://user-images.githubusercontent.com/104793540/185058506-6282e0a0-2efe-4f03-9759-c9abca7083d1.png)

![image](https://user-images.githubusercontent.com/104793540/185059257-b03c4da8-8879-4d6a-a21c-97c290236eb4.png)

 
 Step 3 - ENV Var DB_HOST
 
 what is the use of ENV Var DB_HOST according to developers and how are you going to use it - which VM will need to have this created
 - As shown in the multi machine diagram above, the DB_HOST env variable is needed for the two virtual machines to commnuicate 
 - Such a varible can be set in the App vim permenantly by following the guide in the link below. 
 - https://phoenixnap.com/kb/linux-set-environment-variable
 
 ![image](https://user-images.githubusercontent.com/104793540/185058691-dfe93b13-daa2-4d5f-838b-77b3717370b9.png)

 - The variable must be writte in the .bashrc file in the home vagrant fodler 
 - `ls -a`
 - `sudo nano .bashrc`
 - `DB_HOST=mongodb://192.168.10.150:27017/posts`
 
The variable is needed only in the app vm because it has the ip for the db machine so app knows where to try to connect.


![image](https://user-images.githubusercontent.com/104793540/185060219-f51c6b07-288a-44c2-ab01-c83f66b54748.png)

### DB_HOST
- steps on diagram
1 install mongodb (required version) & 2 required dependencies 
### required key 
- `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927`
- `echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list`
- sudo update and upgrade 
### range of versions 
- `sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20`
- `sudo systemctl restart mongod` & `sudo systemctl enable mongod`
- `sudo systemctl status mongod`

3 edit mongodb file cd/etc/mongod.conf
`cd /etc`
`sudo nano  mongod.conf`
go to network interfaces change to 0.0.0.0 - cant do this in production 

need to tell mongodb you will receieve ip from app please allow 

4 allow app to connect to DB
5 Restart db then enable (systemctl…)
`sudo systemctl restart mongod`
`sudo systemctl enable  mongod`
`sudo systemctl status  mongod`

6 go back to app machine and relaunch app 
`export DB_HOST=mongodb://192.168.10.150:27017/posts`
`printenv DB_HOST`  
`cd app`
`cd app`
`npm start`
`cd seeds`
`node seed.js`
`cd ..`
`npm start`
Refresh browser - Open browser app-ip/posts or ip:3000/posts 

note > Save and exit the file. The changes are applied after you restart the shell. If you want to apply the changes during the current session, use the source command:
`source ~/.bashrc`

only manual things should be 
vagrant up ssh app
cd app/app
npm start
