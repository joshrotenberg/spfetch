# spfetch  (Redis Enterprise Cluster Auditing Tool)

----------


spfetch is a utility for quickly downloading Redis Enterprise support packages and generating audit reports for Redis Enterprise Software deployments.  

spfetch accesses a Redis Enterprise Cluster using the Redis Enterprise Software API.  This requires a valid Redis Enterprise user and password for each cluster to be used.  This can be supplied explicitly on the command line but using the credential vault is recommended.  

credstore is a companion utility that works with spfetch to manage credentials. credstore encrypts and stores credentials that can be retrieved and used by spfetch.  This allows automated scripts to use spfetch in other tooling without having to expose user names and passwords in plain text.  It also provides a mechanism to collect support package and inventory information across multiple clusters using a single command.  
  

	$spfetch.py -h
	usage: spfetch.py [-h] [--user USER] [--pwd PWD] [--path PATH] [--db DB] [--upload] [--nosave] [--keep KEEP [--bloat] [--dryrun] [--list] [--json] [--license] [--xls] fqdn

	Redis Enterprise Audit Tool Version 0.9

	positional arguments:
		fqdn         Fully Qualified Domain Name of Cluster (wildcards supported if credentials in vault))

	options:
	  -h, --help   show this help message and exit
	  --user USER  Username for authentication
	  --pwd PWD    Password for Authentication
	  --path PATH  Folder path for saving Output Files
	  --db DB      Database Id
	  --upload     Upload Package to Redis.io (requires API KEY)
	  --nosave     Do not save support package to disk (works only with --upload
	  --keep KEEP  Number of Output files to keep
	  --bloat      Make no attempt to reduce the Package Size
	  --dryrun     Do a dryrun for testing
	  --list       List Databases Id and Names
	  --json       Format database output list in Json
	  --license    List Databases Id and Names
	  --xls        Generate Excel Inventory Report For All Clusters`



----------

## Quick Start


1. Ensure you have python 3.8 or higher

1. Ensure you have git installed. 

1. Install the following dependencies. 

    	pip install requests
		pip install keyring
    	pip install cryptography
    	pip install openpyxl
    	pip install Files.com
    
1.  Retrieve the code from Github

	git pull https://github.com/redisjohn/spfetch
    
1. Initialize the Credential Vault 

	    credstore.py init 

1. Add cluster credentials to the vault. 

    	credstore.py add --fqdn cluster1.mydomain.com --user redisuser --pass mypassword

1.	To fetch a support Package

		spfetch.py cluster1.mydomain.com
	
1.  The packages are located in the "output" folder.  

----------

## Deep Dive

#### Requirements for Redis Enterprise User Permission

The user credentials provided must have a Management Role of `DB Member` or higher to access the Redis Software API endpoints used by spfetch.    

#### Explicitly Providing Credentials

The only positional argument required to spfetch is a cluster fully qualified domain name.  If the credential vault has not been initialized or the cluster has not been added to the vault, a user name and password is required.   

    $spfetch.py  cluster.redis.test --user test@myorg.com --pwd mypassword 

    2024-10-28 11:49:03,spfetch,INFO,(cluster.redis.test):Agressively Optimizing Support Package Size
    2024-10-28 11:49:03,spfetch,INFO,(cluster.redis.test):Starting Download
	2024-10-28 11:50:18,spfetch,INFO,(cluster.redis.test):Reducing Package Size
	2024-10-28 11:50:32,spfetch,INFO,(cluster.redis.test):Original tar size: 214136954 bytes
	2024-10-28 11:50:32,spfetch,INFO,(cluster.redis.test):New tar size: 2039395 bytes
	2024-10-28 11:50:32,spfetch,INFO,(cluster.redis.test):Storage savings: 212097559 bytes
	2024-10-28 11:50:32,spfetch,INFO,(cluster.redis.test):Support package downloaded successfully.
	2024-10-28 11:50:32,spfetch,INFO,Purging Old Version:(output\debuginfo.cluster.redis.test_20241028112421.tar.gz)   

#### Using the Credential Vault

In order to avoid exposing user name and passwords, spfetch supports adding cluster credentials to a encrypted vault.  Using this feature also allows the convenience of operating on multiple clusters with one command using wild cards for the fqdn.  If no password is provided, then spfetch will automatically check the credentials vault and automatic retrieve the credentials for the specified cluster.  

  	$spfetch.py cluster.redis.test

To configure the credential vault:

The vault must be initialized before it can be used to store credentials

	$credstore.py init 

Clusters can be added to the vault using the add command:

	$crestore.py add --fqdn cluster.redis.test bob@myorg.com mypassword

spfetch uses the Redis Enterprise Software REST API ([https://redis.io/docs/latest/operate/rs/references/rest-api/](https://redis.io/docs/latest/operate/rs/references/rest-api/)).   Authentication to this API is done using BASIC AUTH.   The credstore utility is used to encrypt and store user name and passwords that can be retrieved as needed without exposing them in plain text.  

#### Using Wild Cards with spfetch

If the credential vault is initalized and populated, wild cards can be used with spfetch to select a subset of FQDNs.  This can be used with any spfetch operation to process requests against multiple clusters. 

Some examples of supported wild carding.

   	rlec*
	* 
    *dc1*
    *.net 


#### Batch processing Support Packages Downloads

It is easy to use wild cards to download multiple support packages at once.  
    
    $spfetch.py *.ixaac.net   
    2024-10-28 11:53:54,spfetch,INFO,(rlec1.ixaac.net):Agressively Optimizing Support Package Size
    2024-10-28 11:53:54,spfetch,INFO,(rlec1.ixaac.net):Starting Download
    2024-10-28 11:55:09,spfetch,INFO,(rlec1.ixaac.net):Reducing Package Size
    2024-10-28 11:55:22,spfetch,INFO,(rlec1.ixaac.net):Original tar size: 214159442 bytes
    2024-10-28 11:55:22,spfetch,INFO,(rlec1.ixaac.net):New tar size: 2041851 bytes
    2024-10-28 11:55:22,spfetch,INFO,(rlec1.ixaac.net):Storage savings: 212117591 bytes
    2024-10-28 11:55:22,spfetch,INFO,(rlec1.ixaac.net):Support package downloaded successfully.
    2024-10-28 11:55:22,spfetch,INFO,Purging Old Version:(output\debuginfo.rlec1.ixaac.net_20241028111754.tar.gz)
    2024-10-28 11:55:22,spfetch,INFO,(rlec2.ixaac.net):Agressively Optimizing Support Package Size
    2024-10-28 11:55:22,spfetch,INFO,(rlec2.ixaac.net):Starting Download
    2024-10-28 11:56:37,spfetch,INFO,(rlec2.ixaac.net):Reducing Package Size
    2024-10-28 11:56:50,spfetch,INFO,(rlec2.ixaac.net):Original tar size: 214176922 bytes
    2024-10-28 11:56:50,spfetch,INFO,(rlec2.ixaac.net):New tar size: 2043745 bytes
    2024-10-28 11:56:50,spfetch,INFO,(rlec2.ixaac.net):Storage savings: 212133177 bytes
    2024-10-28 11:56:50,spfetch,INFO,(rlec2.ixaac.net):Support package downloaded successfully.
    2024-10-28 11:56:50,spfetch,INFO,Purging Old Version:(output\debuginfo.rlec2.ixaac.net_20241028111922.tar.gz)
	2024-10-28 11:56:50,spfetch,INFO,(rlec3.ixaac.net):Starting Download
	2024-10-28 11:58:04,spfetch,INFO,(rlec3.ixaac.net):Reducing Package Size
	2024-10-28 11:58:18,spfetch,INFO,(rlec3.ixaac.net):Original tar size: 214189015 bytes
	2024-10-28 11:58:18,spfetch,INFO,(rlec3.ixaac.net):New tar size: 2044389 bytes
	2024-10-28 11:58:18,spfetch,INFO,(rlec3.ixaac.net):Storage savings: 212144626 bytes
	2024-10-28 11:58:18,spfetch,INFO,(rlec3.ixaac.net):Support package downloaded successfully.
	2024-10-28 11:58:18,spfetch,INFO,Purging Old Version:(output\debuginfo.rlec3.ixaac.net_20241028112621.tar.gz)    

#### Pull Support Packages for a Single Database

You can use the --db flag to pull a support package for a single database.  This helps reduce the size of the support package.  

	$spfetch.py --db 1  cluster.redis.test 

You can get a list of database ids using the following command:

	$spfetch.py --list cluster.redis.test

#### Optimization of support package size

spfetch by default will attempt to optimize a support package output by trimming log files.  This is the default behavior.  To bypass optimization using the `--bloat ` flag.


#### Overriding support package download location

spfetch by default will store all output generated in the output folder under the main spfetch directory.  To override this location use the `--path` flag.  

#### Purging old files

spfetch will remove all old versions of a support packages by default each time is generates an updated copy.  You can override the number of old version using the `--keep ` flag.   For inventory reports, the default value is 5. 

    $spfetch.py rlec4.ixaac.net --keep 3
    $spfetch.py --xls * --keep 2 

    
#### Auditing Deployments

spfetch provides several other functions to audit cluster deployments. 

The license command provides a quick list of license information for one or more clusters. 

    $spfetch.py --license *
    {
    "cluster": "cluster.redis.test",
    "expired": true,
    "expiration": "2024-10-01",
    "shards_limit": 500,
    "ram_shards": 3,
    "flash_shards": 0
    }
    {
    "cluster": "rlec1.ixaac.net",
    "expired": false,
    "expiration": "2025-07-01",
    "shards_limit": 10,
    "ram_shards": 9,
    "flash_shards": 0
    }
    {
    "cluster": "rlec2.ixaac.net",
    "expired": false,
    "expiration": "2025-07-01",
    "shards_limit": 35,
    "ram_shards": 35,
    "flash_shards": 0
    }
    {
    "cluster": "rlec3.ixaac.net",
    "expired": false,
    "expiration": "2025-10-01",
    "shards_limit": 20,
    "ram_shards": 0,
    "flash_shards": 10
    }


#### Getting database inventory for a cluster. 

To get a quick summary of databases in a cluster.  (You can also use wild cards with this command for any clusters with credentials stored in the vault)

    $spfetch.py --list cluster.redis.test

	cluster.redis.test
	Id  Name    Version Shards
	--------------------------
	1    test    7.2.3     2
	2    test2   7.2.3     1
    

To get a more detailed summary using the --json flag 

		$spfetch.py --list --json cluster.redis.test 
		{
	    "cluster": "cluster.redis.test",
	    "databases": [
	        {
	            "Id": 1,
	            "Name": "test",
	            "Version": "7.2.3",
	            "Total Shards": 2,
	            "High Availability": true,
	            "Flex": false,
	            "Created": "2024-05-14",
	            "Modified": "2024-09-03",
	            "Persistence": "disabled",
	            "Eviction Policy": "volatile-lru",
	            "CRDB": false,
	            "Modules": "",
	            "TLS": true,
	            "OSS Cluster API": false,
	            "Proxy Policy": "single",
	            "Shard Placement": "dense"
	        },
	        {
	            "Id": 2,
	            "Name": "test2",
	            "Version": "7.2.3",
	            "Total Shards": 1,
	            "High Availability": false,
	            "Flex": false,
	            "Created": "2024-09-20",
	            "Modified": "2024-09-20",
	            "Persistence": "disabled",
	            "Eviction Policy": "volatile-lru",
	            "CRDB": false,
	            "Modules": "",
	            "TLS": false,
	            "OSS Cluster API": false,
	            "Proxy Policy": "single",
	            "Shard Placement": "dense"
	        }
	    ]
	}

### Upload Packages to Redis

In addition to downloading support packages, you can also upload support packages to Redis.  To upload a support package, an API key is required.   Please contact your Redis Enterprise Account Representative to obtain an API key.  Once you have an API key, it must be stored in the credentials vault.  

      $credstore.py apikey --key 04040403828489492302ac93d934c9c934c9c349

To upload a support package you can use the `--upload ` flag.  

	  $spfetch.py --db 1 --upload  cluster.redis.test

If you do not want to save the support package to disk, you can use the `--nosave` flag. 

      $spfetch.py --db 1 --upload --nosave cluster.redis.test     

----------

### Generating An Inventory Report of all Deployments

spfetch can generate an up to date inventory of all clusters deployed.  The inventory report is formatted as a multi-sheet excel workbook with multiple sheets. 

The cluster tab includes a list of all clusters.  A tab is created for each cluster showing detailed information including version numbers of database and modules along with other key settings. 

    spfetch.py --xls *
	2024-10-28 12:26:32,spfetch,INFO,Processing Data for:(cluster.redis.test)
	2024-10-28 12:26:32,spfetch,INFO,Processing Data for:(rlec1.ixaac.net)
	2024-10-28 12:26:32,spfetch,INFO,Processing Data for:(rlec2.ixaac.net)
	2024-10-28 12:26:32,spfetch,INFO,Processing Data for:(rlec3.ixaac.net)
	2024-10-28 12:26:32,spfetch,INFO,Workbook saved as 'output\inventory_20241028122632.xlsx'.
	2024-10-28 12:26:32,spfetch,INFO,Purging Old Version:(output\inventory_20241028094251.xlsx)



----------

## credstore.py Credentials Management
    
    usage: credstore.py [-h] {init,add,get,recover,apikey,reset} ...

	Redis Enterprise Cluster Credentials Encrypted Store

	positional arguments: {init,add,get,recover,apikey,reset}

                        Available commands
    init                Initialize the vault
    add                 Add encrypted credentials for a cluster
    get                 Get Credentials for a cluster
    recover             Recovery Vault from Secret
    apikey              Save Upload API Key
    reset               Delete Vault Secret

	options:
  	  -h, --help            show this help message and exit


credstore.py can be used to manage credentials. It implements a password vault to save all databases credentials. The credentials are encrypted using Fernet, a 128 Bit Symmetric Block Cipher. 

The file CredentialVault.py can be used to update the Cipher if required by replacing the `encrypt_credentials` and `decrypt_credentials` methods. 

Encrypted Credentials for each cluster are store in a file in the vault folder.  

The secret key for the vault is stored in the system key chain.  If the vault needs to be relocated to another machine, the vault can be moved and reused as by inserting the secret into the key chain of the new system. 


#### Managing Credentials with credstore 

The credstore utility is used to manage credentials.  

To initialize the vault run the following command:

	credstore.py init

You should save the secret in a secure location.  This will allow you to move the credentials vault to another system if needed in the future.  

Once the vault is initialized you can store encrypted credentials for each cluster. 

	credstore.py add {fqdn} --user {username} --pwd {password} 


#### Removing a cluster from the vault

To remove a cluster from the vault, go the vault folder and delete the file that corresponds with the fqdn you 
wish to remove.   

#### Moving the Credential Vault to another System

To restore the vault to a different system, you must have a backup of the vault directory and have the secret key generated when the vault was initiated.   After the vault directory is restored on the new system, run the following command:

	credstore.py recover {secretkey}  






