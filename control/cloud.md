---
layout: page
title: "CLI Reference"
keywords: portworx, pxctl, command-line tool, cli, reference
sidebar: home_sidebar
redirect_from: "/cli-reference.html"
---

* TOC
{:toc}

### Cloud operations
Help for specific cloudsnap commands can be found by running the following command

Note: All cloudsnap operations requires secrets login to configured endpoint with/without encryption. Please refer pxctl secrets cmd help.
#### pxctl cloudsnap --help
```
sudo /opt/pwx/bin/pxctl cloudsnap --help
NAME:
   pxctl cloudsnap - Backup and restore snapshots to/from cloud

USAGE:
   pxctl cloudsnap command [command options] [arguments...]

COMMANDS:
     backup, b          Backup a snapshot to cloud
     restore, r         Restore volume to a cloud snapshot
     list, l            List snapshot in cloud
     status, s          Report status of active backups/restores
     schedule, sc       Update cloud-snap schedule
     credentials, cred  Manage cloud-snap credentials

OPTIONS:
   --help, -h  show help
```

#### pxctl cloudsnap credentials
This command is used to create/list/validate/delete the credentials for cloud providers. These credentials will be used for cloudsnap of volume to the cloud.  

Note: It will create a bucket with the portworx cluster ID to use for the backups
```
sudo /opt/pwx/bin/pxctl cloudsnap credentials
NAME:
   pxctl cloudsnap credentials - Manage cloud-snap credentials

USAGE:
   pxctl cloudsnap credentials command [command options] [arguments...]

COMMANDS:
     create, c    Create a credential for cloud-snap
     list, l      List all credentials for cloud-snap
     delete, d    Delete a credential for cloud-snap
     validate, v  Validate a credential for cloud-snap

OPTIONS:
   --help, -h  show help
```

#### pxctl cloudsnap credentials list
`pxctl cloudsnap credentials list` is used to list all configured credential keys
```
sudo /opt/pwx/bin/pxctl cloudsnap credentials list

S3 Credentials
UUID						REGION			ENDPOINT			ACCESS KEY			SSL ENABLED	ENCRYPTION
ffffffff-ffff-ffff-1111-ffffffffffff		us-east-1		s3.amazonaws.com		AAAAAAAAAAAAAAAAAAAA		false		false

Azure Credentials
UUID						ACCOUNT NAME		ENCRYPTION
ffffffff-ffff-ffff-ffff-ffffffffffff		portworxtest		false
```

#### pxctl cloudsnap credentials create
`pxctl cloudsnap credentials create` is used to create/configure credentials for various cloud providers
```
sudo /opt/pwx/bin/pxctl cloudsnap cred create --provider s3 --s3-access-key AAAAAAAAAAAAAAAA --s3-secret-key XXXXXXXXXXXXXXXX --s3-region us-east-1 --s3-endpoint s3.amazonaws.com
Credentials created successfully
```

#### pxctl cloudsnap credentials delete
`pxctl cloudsnap credentials delete` is used to delete the credentials from the cloud providers.
```
sudo /opt/pwx/bin/pxctl cloudsnap cred delete --uuid ffffffff-ffff-ffff-1111-ffffffffffff
Credential deleted successfully
```

#### pxctl cloudsnap credentials validate
`pxctl cloudsnap credentials validate` validates the existing credentials
```
sudo /opt/pwx/bin/pxctl cloudsnap cred validate --uuid ffffffff-ffff-ffff-1111-ffffffffffff
Credential validated successfully
```

#### pxctl cloudsnap backup
`pxctl cloudsnap backup` command is used to backup a single volume to the configured cloud provider through credential command line. 
If it will be the first backup for the volume a full backup of the volume is generated. If it is not the first backup, it only generates an incremental backup from the previous full/incremental backup.
If a single cloud provider credential is created then there is no need to specify the credentials on the command line.
```
sudo /opt/pwx/bin/pxctl cloudsnap backup vol1
Cloudsnap backup started successfully
```
If multiple cloud providers credentials are created then need to specify the credential to use for backup on command line
```
sudo /opt/pwx/bin/pxctl cloudsnap backup vol1 --cred-uuid ffffffff-ffff-ffff-1111-ffffffffffff 
Cloudsnap backup started successfully
```
Note: All cloudsnap backup/Restores can be monitored through CloudSnap status command which is described in following sections

#### pxctl cloudsnap restore
`pxctl cloudsnap restore` command is used to restore a successful backup from cloud. (Use cloudsnap list command to get the cloudsnap Id). It requires cloudsnap Id (to be restored) and credentials. 
Restore happens on any node in the cluster where storage can be provisioned. In this release, restored volume will be of replication factor 1. 
This volume can be updated to different repl factors using volume ha-update command.
```
sudo /opt/pwx/bin/pxctl cloudsnap restore --snap gossip12/181112018587037740-545317760526242886
Cloudsnap restore started successfully: 315244422215869148
```
Note: All cloudsnap backup/Restores can be monitored through CloudSnap status command which is described in following sections

#### pxctl cloudsnap status
`pxctl cloudsnap status` can be used to check the status of cloudsnap operations
```
sudo /opt/pwx/bin/pxctl cloudsnap status
SOURCEVOLUME		STATE		BYTES-PROCESSED	TIME-ELAPSED		COMPLETED			          ERROR
1040525385624900824	Restore-Done	11753581193	8m32.231744596s		Wed, 05 Apr 2017 06:57:08 UTC
1137394071301823388	Backup-Done	11753581193	1m46.023734966s		    Wed, 05 Apr 2017 05:03:42 UTC
13292162184271348	Backup-Done	27206221391	4m25.740022954s		    Wed, 05 Apr 2017 22:39:41 UTC
454969905909227504	Backup-Active	91944386560	4h8m19.283242837s
827276927130532677	Restore-Failed	0									             Failed to authenticate creds ID
```

#### pxctl cloudsnap list
`pxctl cloudsnap list` is used to list all the cloud snapshots
```
sudo /opt/pwx/bin/pxctl cloudsnap list --cred-uuid ffffffff-ffff-ffff-1111-ffffffffffff --all
SOURCEVOLUME 			CLOUD-SNAP-ID									CREATED-TIME				STATUS
vol1			gossip12/181112018587037740-545317760526242886		Sun, 09 Apr 2017 14:35:28 UTC		Done
```
Filtering on cluster ID or volume ID is available and can be done as follows:
```
sudo /opt/pwx/bin/pxctl cloudsnap list --cred-uuid ffffffff-ffff-ffff-1111-ffffffffffff --src vol1
SOURCEVOLUME 		CLOUD-SNAP-ID					CREATED-TIME				STATUS
vol1			1137394071301823388-283948499973931602		Wed, 05 Apr 2017 04:50:35 UTC		Done
vol1			1137394071301823388-674319852060841900		Wed, 05 Apr 2017 05:01:56 UTC		Done

sudo /opt/pwx/bin/pxctl cloudsnap list --cred-uuid ffffffff-ffff-ffff-1111-ffffffffffff --cluster cs25
SOURCEVOLUME 		CLOUD-SNAP-ID					CREATED-TIME				STATUS
vol1			1137394071301823388-283948499973931602		Wed, 05 Apr 2017 04:50:35 UTC		Done
vol1			1137394071301823388-674319852060841900		Wed, 05 Apr 2017 05:01:56 UTC		Done
volshared1		13292162184271348-457364119636591866		Wed, 05 Apr 2017 22:35:16 UTC		Done
```
