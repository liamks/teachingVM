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

### Step 1: In VirtualBox select TeachingVM

![Select TeachingVM](/screenshots/shared-folders/1-virtualbox.png?raw=true "Select TeachingVM")

### Step 2: Click Settings

![Settings](/screenshots/shared-folders/2-settings.png?raw=true "Settings")

### Step 3: Click Shared Folders

![Shared Folders](/screenshots/shared-folders/3-shared-folders.png?raw=true "Shared Folders")

## Step 4: Click the + folder to add a new shared folder

![Add Shared Folders](/screenshots/shared-folders/4-add-shared-folder.png?raw=true "Add Shared Folder")

* Folder Path: Select your Documents folder
* Folder Name: `/home/vagrant/documents`
* Click Ok!

## Connecting to your VM

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
rake db:migrate
rake db:setup
```

### Running Ruby on Rails

```
bin/rails server -b 0.0.0.0
```