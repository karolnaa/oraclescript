# Oracle Free Tier Instance Creation Through Python

[![Created Badge](https://badges.pufler.dev/created/mohankumarpaluru/oracle-freetier-instance-creation)](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation) [![Updated Badge](https://badges.pufler.dev/updated/mohankumarpaluru/oracle-freetier-instance-creation)](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation) [![Visits Badge](https://badges.pufler.dev/visits/mohankumarpaluru/oracle-freetier-instance-creation)](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation) [![HitCount](https://img.shields.io/endpoint?url=https%3A%2F%2Fhits.dwyl.com%2Fmohankumarpaluru%2Foracle-freetier-instance-creation.svg%3Fstyle%3Dflat%26show%3Dunique%3Fcolor=brightgreen)](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation) [![GitHub stars](https://img.shields.io/github/stars/mohankumarpaluru/oracle-freetier-instance-creation?color=brightgreen)](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation/stargazers)
[![GitHub issues](https://img.shields.io/github/issues/mohankumarpaluru/oracle-freetier-instance-creation?color=brightgreen)](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation/issues) [![GitHub forks](https://img.shields.io/github/forks/mohankumarpaluru/oracle-freetier-instance-creation?color=brightgreen)](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation/network) [![GitHub license](https://img.shields.io/github/license/mohankumarpaluru/oracle-freetier-instance-creation?color=brightgreen)](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation/blob/main/LICENSE) 


This project provides Python and shell scripts to automate the creation of Oracle Free Tier ARM instances (4 OCPU, 24 GB RAM) or the Oracle Free Tier AMD instance (1 OCPU, 1 GB RAM) with minimal manual intervention. Acquiring resources in certain availability domains can be challenging due to high demand, and repeatedly attempting creation through the Oracle console is impractical. While other methods like OCI CLI and PHP are available (linked at the end), this solution aims to streamline the process by implementing it in Python.

The script attempts to create an instance every 60 seconds or as per the `REQUEST_WAIT_TIME_SECS` variable specified in the `oci.env` file until the instance is successfully created. Upon completion, a file named `INSTANCE_CREATED` is generated in the project directory, containing details about the newly created instance. Additionally, you can configure the script to send a Gmail notification upon instance creation.

**Note: This script doesn't configure a public IP by default; you need to configure it post the creation of the instance from the console. (Planning on automating it soon)**

In short, this script is another way to bypass the "Out of host capacity" or "Out of capacity for shape VM.Standard.A1.Flex" error and create an instance when the resources are freed up.

## Features
- Single file needs to be run after basic setup
- Configurable wait time, OCPU, RAM, DISPLAY_NAME
- Gmail notification
- SSH keys for ARM instances can be automatically created
- OS configuration based on Image ID or OS and version
- Compute shape configuration

## Pre-Requisites
- **VM.Standard.E2.1.Micro Instance**: The script is designed for a Ubuntu environment, and you need an existing subnet ID for ARM instance creation. Create an always-free `VM.Standard.E2.1.Micro` instance with Ubuntu 22.04. This instance can be deleted after the ARM instance creation. (Not required if an existing OCI_SUBNET_ID is defined in oci.env file)
- **OCI API Key (Private Key) & Config Details**: Follow this [Oracle API Key Generation link](https://graph.org/Oracle-API-Key-Generation-12-11) to create the necessary API key and config details.
 - Note: Typically the API Key can be generated from your profile [page](https://cloud.oracle.com/identity/domains/my-profile/api-keys) > API Keys (left) > Add API Key
- **OCI Free Availability Domain**: Identify the eligible always-free tier availability domain during instance creation.
- **Gmail App Passkey (Optional)**: If you want to receive an email notification after instance creation and have two-factor authentication enabled, follow this [Google App's Password Generation link](https://graph.org/Google-App-Passwords-Generation-12-11) to create a custom app and obtain the passkey.

## Setup

1. SSH into the VM.Standard.E2.1.Micro Ubuntu machine, clone this repository, and navigate to the project directory. Change the permissions of `setup_init.sh` to make it executable.
    ```bash
    git clone https://github.com/mohankumarpaluru/oracle-freetier-instance-creation.git
    cd oracle-freetier-instance-creation
    ```

2. Create a file named `oci_api_private_key.pem` and paste the contents of your API private key. The name and path of the file can be anything, but the current user should have read access.

3. Create a file named `oci_config` inside the repository directory. Paste the config details copied during the OCI API key creation. Refer to `sample_oci_config`.

4. In your `oci_config`, fill the **`key_file`** with the absolute path of your `oci_api_private_key.pem`. For example, `/home/ubuntu/oracle-freetier-instance-creation/oci_api_private_key.pem`.

5. Edit the **`oci.env`** file and fill in the necessary details. Refer [below for more information](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation#environment-variables) `oci.env` fields.

## Run

Once the setup is complete, run the `setup_init.sh` script from the project directory. This script installs the required dependencies and starts the Python program in the background.
```bash
./setup_init.sh
```
If you are running in a fresh `VM.Standard.E2.1.Micro` instance, you might receive a prompt *Daemons using outdated libraries*. Just click `OK`; that's due to updating the libraries through apt update and won't be asked again. 

If you are running in your local instead of `VM.Standard.E2.1.Micro` instance, make sure you fill the `OCI_SUBNET_ID`. 

The script will display an error prompt if an issue arises; otherwise, it will show "Script is running successfully."

View the logs of the instance creation API call in `launch_instance.log` and details about the parameters used (availability-domain, compartment-id, subnet-id, image-id) in `setup_and_info.log`.

## Errors and Re-Run 

If the `oci_config` file is found to be incorrect, the script generates an `ERROR_IN_CONFIG.log` file. Verify the `oci_config` for accuracy, ensuring it aligns with the [sample_oci_config](https://github.com/mohankumarpaluru/oracle-freetier-instance-creation/blob/85b3ec065a91bb66206933a12a6bd58941446118/sample_oci_config#L1C1-L6C80) without any additional lines or characters.


In case of an unhandled exception leading to script termination, an email containing the logs is sent if opted. Otherwise, only the error logs are printed to `UNHANDLED_ERROR.log`. Review the logs and execute the script again using the following command (which skips dependency installation). If the issue persists, raise an issue with the contents of `UNHANDLED_ERROR.log`.

```bash
./setup_init.sh rerun
```

## TODO
- [ ] Ability to run script locally :
	- [x] By letting user configure existing oracle subnet id in `OCI_CONFIG`.
	- [ ] By creating VPC and subnet from Script if running locally (need to handle the free tier limits).
- [ ] Make Boot Volume Size configurable and handle errors and free tier limits.
- [ ] Assign a public IP through the script and handle free tier limits.
- [ ] Make the script interavtive by displaying a list of images and OS that can be used before launching an instance to select.
- [ ] Redirect logs to a Telegram Bot.

## Environment Variables
**Required Fields:**
	
- `OCI_CONFIG`:  Absolute path to the file with OCI API Config Detail content 
- `OCT_FREE_AD`: Availability Domain that's eligible for *Always-Free Tier*. If multiple, separate by commas

**Optional Fields:**
- `DISPLAY_NAME`: Name of the Instance 
- `REQUEST_WAIT_TIME_SECS`: Wait before trying to launch an instance again.  
- `SSH_AUTHORIZED_KEYS_FILE`: Give the absolute path of an SSH public key for ARM instance. **The program will create a public and private key pair with the name specified if the key file doesn't exist; otherwise, it uses the one specified**.
- `OCI_SUBNET_ID`: The `OCID` of an existing subnet that will be used when creating an ARM instance. Only use it for running script from local. DO NOT ADD THIS IF YOU ARE ALREADY RUNNING IN A MICRO INSTANCE.
    >  This can be found in `Networking` >`Virtual cloud networks` > `<VPC-Name>` > `Subnet Details`.
- `OCI_IMAGE_ID`: *Image_id* of the desired OS and version; the script will generate the `image_list.json`. 
- `OCI_COMPUTE_SHAPE`: Free-tier compute shape of the instance to launch. Defaults to ARM, but configurable if you are running into capacity issues for the free AMD instance in your home region. Acceptable values `VM.Standard.A1.Flex` and `VM.Standard.E2.1.Micro`.
- `SECOND_MICRO_INSTANCE`: `True` if you are utilizing the script for your second free tier Micro Instance, else `False`.
- `OPERATING_SYSTEM`: Exact name of the operating system 
- `OS_VERSION`: Exact version of the operating system 
- `NOTIFY_EMAIL`: Make it True if you want to get notified and provide email and password
- `EMAIL`: Only Gmail is allowed, the same email will be used for *FROM* and *TO*
- `EMAIL_PASSWORD`: If two-factor authentication is set, create an App Password and specify it, not the email password. Direct password will work if no two-factor authentication is configured for the email.

## Credits and References
- [xitroff](https://www.reddit.com/user/xitroff/): [Resolving Oracle Cloud Out of Capacity Issue and Getting Free VPS with 4 ARM Cores, 24GB of RAM](https://hitrov.medium.com/resolving-oracle-cloud-out-of-capacity-issue-and-getting-free-vps-with-4-arm-cores-24gb-of-a3d7e6a027a8)
  - [Github Repo](https://github.com/hitrov/oci-arm-host-capacity)
- [Oracle Launch Instance Docs](https://docs.oracle.com/en-us/iaas/api/#/en/iaas/20160918/Instance/LaunchInstance)
- [LaunchInstanceDetails](https://docs.oracle.com/en-us/iaas/api/#/en/iaas/20160918/datatypes/LaunchInstanceDetails)
