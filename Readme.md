# Teaching VM

## Getting Started (Windows & Mac)

1. Download VirtualBox from: `https://www.virtualbox.org/wiki/Downloads`*
2. Install VirtualBox
3. Download virtual machine ("TeachingVM.ova")*
4. In VirtualBox, import TeachingVM.ova (File -> Import Appliance)
*Alternatively I've brought USB keys with the above.

## Windows Users

1. Download Putty from: `http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html` (Choose the "A Windows installer for everything except PuTTYtel")
2. Install Putty.

Putty is used to SSH into your VM.

## Setting Up Shared folders

### Step 1: Install VBoxGuestAdditions (virtualbox guest additions)
```
sudo mkdir /media/ga
sudo mount -o loop VBoxGuestAdditions_4.3.28.iso /media/ga
cd /media/ga/
sudo ./VBoxLinuxAdditions.run 
```

Above we're creating a folder `ga` and mounting the VBoxGuestAdditions `.iso` to that folder. Then
we go into the folder and install guest additions by running `sudo ./VboxLinuxAdditions.run`.

### Step 2: In VirtualBox select TeachingVM

![Select TeachingVM](/screenshots/shared-folders/1-virtualbox.png?raw=true "Select TeachingVM")

### Step 3: Click Settings

![Settings](/screenshots/shared-folders/2-settings.png?raw=true "Settings")

### Step 4: Click Shared Folders, then + folder to add a new shared folder

![Add Shared Folders](/screenshots/shared-folders/3-add-shared-folder.png?raw=true "Add Shared Folder")

* Folder Path: Select your Documents folder (or what ever folder you want to access in the VM)
* Folder Name: `docs`
* Click Ok!
* Restart your VM (right click on VM, close > Power Off, then start it again)

## Step 5: Mounting the shared folder into the VM

We've installed guest additions, and selected the folder we want to share, now we need to mount it! Run the following command:

```
sudo mount -t vboxsf docs /home/vagrant/documents/ -o rw,uid=1000,gid=1000
```

Unfortunately, if you restart you're VM you'll have to run that command again, to fix that edit `/etc/rc.local` by adding `mount -t vboxsf docs /home/vagrant/documents/ -o rw,uid=1000,gid=1000` above the `exit 0` line.


## Connecting to your VM

While you can connect to yoru VM through the window created by VirtualBox the experience is usually unpleasant! I recommend SSH'ing into the machine via your terminal (Mac) or Putty (windows).

### Mac 

1. Open termal
2. Enter: `ssh -p 2202 vagrant@127.0.0.1`

### Windows

1. Open Putty
2. Fill in the following
  1. Host Name (or IP address) = `127.0.0.1`
  2. Port = `2202` (If that's wrong, it should in your VM's options)
  3. [Optional] In saved Sessions enter "TeachingVM", click save
  4. Click "Open" to start the SSH session

![Connect to VM](/screenshots/putty.png?raw=true "Connect to VM")

## TeachingVM Contains

### Languages & Frameworks
- Ruby 2.2.1
- Node 0.12.2
- RoR 4.2.1

### Persistence
- Postgres 9.3 + postgis 2.1
- Mongodb 3.0.2
- Redis 3.0.0

### Misc
- Git
- ImageMagick

## Advanced
### Starting VM with Vagrant

```
vagrant up
```

### Provisioning with Vagrant & Ansible
```
vagrant provision
```

### Creating Rails app with Postgres

#### Step 1: Create App
```
rails new yourappname --database=postgresql
```

#### Step 2: Update Database config
 Edit config/database.yml. You'll have the add the username, password and host lines.

 ```yml
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see rails configuration guide
  # http://guides.rubyonrails.org/configuring.html#database-pooling
  pool: 5
  username: rails
  password: rails
  host: localhost
 ```

### Step 3: run these commands
```
rake db:setup
rake db:migrate
```

### Running Ruby on Rails

```
bin/rails server -b 0.0.0.0
```

### Viewing your Ruby on Rails App

On your machine (not the VM) open a browser and go to `http://127.0.0.1:3001/`.

### View your NodeJS App

Once you've started your app go to `http://127.0.0.1:5001/`
