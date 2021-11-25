---
description: >-
  Below are instructions to use iCommands in an Rstudio VM in Research Cloud. It
  starts from the point where you started an "RStudio with icommands" workspace
  and logged into the VM.
---

# Research Cloud - Yoda install

## Step 1: configure iCommands

In the left panel of RStudio, click on Terminal (next to Console)

{% hint style="info" %}
:mage: **Tips: icommands is pre-installed in the "RStudio with iRods Icommands" application. **If you are using a VM without icommands pre-installed, you can install icommands using the terminal. For instructions see [https://yoda.uu.nl/yoda-uu-nl/getting-started/icommands.html](https://yoda.uu.nl/yoda-uu-nl/getting-started/icommands.html) Copy and paste the lines under "Installing iCommands on Ubuntu" from this website into the terminal.&#x20;
{% endhint %}

Then create an irods configuration file as follows:

```
mkdir .irods
touch .irods/irods_environment.json
nano .irods/irods_environment.json
```

With the `nano` command you will start the na text editor and open the empty file "irods\_environment.json"

Now go the the above mentioned [website](https://yoda.uu.nl/yoda-uu-nl/getting-started/icommands.html) and copy/paste the contents of the configuration file belonging to your faculty in the empty file. The lines look like this:

```
{ 
"irods_host": "i-lab.data.uu.nl", 
"irods_port": 1247, 
"irods_home": "/nluu5p/home", 
"irods_user_name": "exampleuser@uu.nl", 
"irods_zone_name": "nluu5p", 
"irods_authentication_scheme": "pam", 
"irods_encryption_algorithm": "AES-256-CBC", 
"irods_encryption_key_size": 32, 
"irods_encryption_num_hash_rounds": 16, 
"irods_encryption_salt_size": 8, 
"irods_client_server_negotiation": "request_server_negotiation" 
}
```

After pasting, change the user name to your UU email address.

Press `Ctrl + x` to exit the text editor, press `Y` to save when requested followed by Enter.

Now you are ready to connect to Yoda.

Type `iinit` to connect, followed by your solis password.

Further documentation about usage of icommands can be found here: ​[https://docs.irods.org/master/icommands/user/](https://docs.irods.org/master/icommands/user/)​

## Step 2: Get data from Yoda to Research cloud <a href="step-2-get-data-from-yoda-to-research-cloud" id="step-2-get-data-from-yoda-to-research-cloud"></a>

In the terminal window in RStudio, create a data folder in the scratch folder:

```
cd ~/data/volume_1
mkdir inputdata
cd inputdata
```

Navigate to the dataset on Yoda with the `icd` command (tip: use `ils` to list the available folder and files in your current Yoda directory):

```
icd research-<projectname> 
icd data
```

Copy the data to Research cloud

```
iget <dataset>.csv
```

​

{% hint style="info" %}
Tip: if you need to copy all contents of a folder, read up on the `irsync` command on [https://docs.irods.org/master/icommands/user/](https://docs.irods.org/master/icommands/user/)

But...
{% endhint %}

{% hint style="danger" %}
Be careful! If you use the `irsync` command in the "wrong" direction it is possible to delete data. To prevent this, create a backup folder and store your data there. Furthermore create a clean workflow by using separate input folders and output folders.&#x20;

1. Create a "data" folder both on Yoda and on Research Cloud&#x20;
2. Create an "output" folder both on Yoda and on Research Cloud
3. Only copy data from Yoda's "data" folder to Research Clouds "data" folder and only from Research Clouds' "output' folder to Yoda's "output" folder. In this way you reduce the risk of losing data by e.g. accidentally overwriting input data on Yoda.
{% endhint %}

## Step 3: Create script and read data into R <a href="step-3-read-data-into-r" id="step-3-read-data-into-r"></a>

Now first create a folder for your R scripts

```
cd ~/data/volume_1
mkdir R
```



In the left panel of research cloud go to the console tab.

```
setwd('./data/volume_1/R/')
df <- read.csv('./data/volume_1/inputdata/<dataset>.csv')
```
