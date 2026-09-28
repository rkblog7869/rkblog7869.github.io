Backup

Periodic remote backups can be configured in VCF OPS. This automatically translates to Backup Configuration object in Avi which pushes the encrypted full system configuration to the remote server periodically.

This config can be used to restore the system in case of any disaster.

VCF OPS back up config -

This automatically translates to the following in Avi -

How to manually take a backup on a Avi controller - 

POST api/configuration/export?full_system=true
{
    "passphrase": "VMware123!VMware123!"
}
Restore

Note: These steps were tested on a federation testbed with ALB deployed on management domain

Delete the existing controller from VCF OPS UI. If the delete does not go through use query parameter force=True
Fresh deploy the same version controller using VCF OPS with the same parameters as the original deployment.
Once the new deployment is completed, make the controller cluster leader a single node by removing other nodes in controller UI.
Now export the configuration from the new controller via the following API - 

POST api/configuration/export
{
    "passphrase": "VMware123!VMware123!"
}
Save the configuration to a file. Keep only the following objects - 
BackupConfiguration
CloudConnectorUser
META
PKIProfile
SSLKeyAndCertificate
VMCA portal cert
VMCA portal CA
User
avi-um-user
If Avi is in mgmt domain then nsxt-infra-admin else NA
Copy the backup configuration from the remote backup server and run the following in CLI
restore configuration file /home/admin/backup-config.json passphrase <backup passphrase> skip_warnings
This will initiate an "upgrade" on the controller.
Once the restore is completed, Import the pruned exported configuration in Step #5 using the following API - 
POST api/configuration/import
{
    "configuration": {
     <pruned exported configuration>
    },
    "passphrase": "VMware123!VMware123!"
}
Edit systemconfiguration, Change the portal certificate to point to new imported certificate, delete the older certificate.
Add the 2 controller nodes back in the cluster.

Now wait for a few mins for things to reconcile.

To verify if NSX is able to connect to Avi check v1/alb-clusters API from SDDC M, it should show Avi version and cluster status.

 

In Avi the cloud should be in READY state and SEs should reconnect - 

If the cloud isn't in READY state then check Cloud connector users, if they are not able to login to NSX and VC, Reset the passwords using the individual appliance APIs.




This procedure is validation on Cycle 5 VCF build.
Environment:  WCP + VKS

Remote Server Backup - backup.json

After new alb deployment, export configuration and modified json - updated 




Screenshots
























