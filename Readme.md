# Teaching VM

## Getting Started

1. Download VirtualBox from: `https://www.virtualbox.org/wiki/Downloads`*
2. Install VirtualBox
3. Download virtual machine ("TeachingVM.ova")*
4. In VirtualBox, import TeachingVM.ova (File -> Import Appliance)

*Alternatively I've brought USB keys with the above.

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