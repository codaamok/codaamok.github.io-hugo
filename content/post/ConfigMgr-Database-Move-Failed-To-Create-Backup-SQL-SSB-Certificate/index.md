---
title: "ConfigMgr Database Move: Failed to Create/Backup SQL SSB Certificate"
date: 2018-10-15T00:00:00+01:00
draft: false
image: images/cover.jpg
categories:
    - ConfigMgr
---

In a recent project I had to move the SQL database for a single primary site hierarchy to a different host in order to go beyond version 1610, because support for Windows Server 2008 R2 had ended from 1702 onward.

Everything was going smoothly up until it came to modifying the SQL configuration of the site. I was getting slapped by ConfigMgrSetup.log with:

> Create\_BackupSQLCert : SQL server failed to backup cert. CSiteControlSetup::SetupCertificateForSSB : Failed to create/backup SQL SSB certificate. ERROR: Failed to set up SQL Server certificate for service broker on "sccm.contoso.com" .

After a couple of days researching and help from colleagues, I learnt more about MS SQL and SQL certificates.

I won't overcomplicate this or try to act like I know every minute detail of SQL so I'll cut to the chase.

During installation of SQL, I set a domain service account for the agent, database engine and reporting service. After installation, I changed the service account used for these services. Then I attempted the database migration and stumbled across said error. During my test run in a lab env (as it was my first attempt at SCCM db move) I did not encounter this issue, because I did not play hot potato with service accounts.

According to the links below, internal SQL certificates are generated with a dependency on the database's master key. Changing the domain user to run the service meant that its service key could no longer access or decrypt the database's master key.

There are methods to recreate the database master keys detailed in the links below, however after initially trying (and failing) to use the stored procedure **spCreateandBackupSQLCert,** I figured a reinstall of a new SQL instance using the desired service account would do the trick. The penny dropped when I read the below from [here](https://blogs.technet.microsoft.com/umairkhan/2013/12/12/configmgr-2012-drs-and-sql-service-broker-certificate-issues/):

> The history was the SQL server was installed with the system account and then later changed to a domain user account.
> 
> The problem with doing the above is that when Configuration Manager is installed it creates some internal certificates which are dependent on the master key. When the account being used to run the database server changes the new account is no longer able to 'unlock' the master key and consequently can not read the internal certificates which then cause communication between sites to fail.

In case you didn't know, because I didn't, SQL certificates are different to the certificates you may usually deal with in a user's, computer's or service's personal / trusted root store from the Certificates snap in.

Resources I used to help me understand SQL certificates:

- [https://blogs.technet.microsoft.com/umairkhan/2013/12/12/configmgr-2012-drs-and-sql-service-broker-certificate-issues/](https://blogs.technet.microsoft.com/umairkhan/2013/12/12/configmgr-2012-drs-and-sql-service-broker-certificate-issues/)
- [http://www.sqlservercentral.com/articles/Encryption/108693/](http://www.sqlservercentral.com/articles/Encryption/108693/)
- [https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/encryption-hierarchy?view=sql-server-2017](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/encryption-hierarchy?view=sql-server-2017)

Resources I used to help me with the database move:

- [https://blogs.technet.microsoft.com/configurationmgr/2013/04/02/how-to-move-the-configmgr-2012-site-database-to-a-new-sql-server/](https://blogs.technet.microsoft.com/configurationmgr/2013/04/02/how-to-move-the-configmgr-2012-site-database-to-a-new-sql-server/)
- [https://deploymentresearch.com/Research/Post/646/Moving-the-ConfigMgr-Current-Branch-database-to-another-server-as-in-back-to-the-primary-site-server](https://deploymentresearch.com/Research/Post/646/Moving-the-ConfigMgr-Current-Branch-database-to-another-server-as-in-back-to-the-primary-site-server)

