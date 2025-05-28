
## Hadoop Security

**The Security problem:**
1. One can impersonate as someone else to access the user.
2. Only user with the root privileges can impersonate the other user.*a common users are not allowed to impersonate other users*

*Note-HDFS users are resolved only on the master node.*

Hadoop user can perform any command as it is inside the **/etc/sudoers**
```
cat /etc/sudoers
## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment                                                                                )
#includedir /etc/sudoers.d
Defaults:hadoop   !requiretty
hadoop  ALL=(ALL) NOPASSWD:ALL
Defaults:hadoop   !requiretty
Defaults   env_keep += "JAVA_HOME"
```

Hdfs hadoop user are the root user in hdfs-site.xml:
````
 <property>
    <name>dfs.permissions.superusergroup</name>
    <value>hadoop</value>
    <description>The name of the group of super-users.</description>
  </property>
````
````
[jonny@ip-10-X-X-X hadoop]$ export HADOOP_USER_NAME=hadoop-->
````
This has precedence on whoami
```
[jonny@ip-10-16-112-19 hadoop]$ hdfs dfs -ls /
Found 3 items
drwxrwxrwt   - hdfs hadoop          0 2025-01-28 20:12 /tmp
drwxr-xr-x   - hdfs hadoop          0 2025-01-28 20:12 /user
drwxr-xr-x   - hdfs hadoop          0 2025-01-28 20:12 /var
[jonny@ip-10-X-X-X hadoop]$
```
```
[jonny@ip-10-16-112-19 hadoop]$ groups jonny
jonny : hadoop
[jonny@ip-10-16-112-19 hadoop]$
```

**Edge node and why it is needed:**
Without authentication and with proper ports any instances can be an edge node
the following process are listening on all the interfaces!!
Https(port 14000)
ResourceManager REST(port 8088)
Hive(port 10000)

**Netwrok security possible solution:**
1. Properly configured the SecurityGroup
2. Do not Launch the cluster on the Public Subnet
3. Do not allow traffic from everywhere
4. Enable kerberos Authentication
5. Enable the block public access feature-- works only at EMR launch

Active Directory
Database system that provides authentication(kerberos),Directory, policy and other services in a windows environment

## Key Kerberos concepts:
KDC --> Authentication server + Ticket granting Server
Principal(users and services) that wants to be authenticated 

The ticket: 
• Once obtained it is kept in cache to don’t provide the password more than one time
has a fixed duration and a maximum renewal time (renew=renew without inserting a password or specifying a keytab file
Enabling Kerberos on Hadoop, all services communicate in a secure way (every service has its ticket/token )
REALM = the domain over which a Kerberos authentication server has the authority to authenticate a user, host or service • (i.e., @TESTDOMAIN.COM, @MYDOMAIN.COM, .
PRINCIPAL = a user or a service that the KDC recognize and authenticate • (i.e., john@TESTDOMAIN.COM, hdfs/ip-10-0-1-244.us-east- 2.compute.internal@TESTDOMAIN.COM)
By default a principal is created for each Hadoop service present in each machine under the cluster KDC REALM: • hdfs/ip-10-0-1-244.us-east-2.compute.internal@TESTDOMAIN.COM
• yarn/ip-10-0-1-244.us-east-2.compute.internal@TESTDOMAIN.COM
Authentication – Kerberos: MIT KDC & kadmin
EMR Master node: 
• service krb5kdc status--The krb5kdc service is the Kerberos version 5 Authentication Service and Key Distribution Center (AS/KDC)
service kadmin status The kadmin service is the Kerberos database administration program.

By default everyone that has a username that ends with /admin has full access to the KDC DB • cat /var/kerberos/krb5kdc/kadm5.acl • */admin *
Tools to manage: kadmin.local (from the master instance), kadmin (from other instances) • sudo kadmin.local
sudo kadmin.local • Authenticating as principal root/admin@TESTDOMAIN.COM with password. • kadmin.local: list_principals • HTTP/ip-10-0-1-244.us-east-2.compute.internal@TESTDOMAIN.COM
How to add users in Kerberos # kadmin.local on the mater node 
kadmin.local: addprinc ishan@TESTDOMAIN.COM • Enter password for principal "ishan@TESTDOMAIN.COM": • Re-enter password for principal "ishan@TESTDOMAIN.COM": • Principal "ishan@TESTDOMAIN.COM" created. • kadmin.local: quit

On the master node we don't have to enter the password
We didn’t have to prompt a password to access the tool. Why? • key_stash_file = /var/kerberos/krb5kdc/stash • The master key stored in the stash file is used. It contains the encrypted version (master_key_type = des3-hmac-sha1) of the KDC admin password we provided launching the cluster. • 

however To manage remotely: • kadmin -p kadmin/admin • Authenticating as principal kadmin/admin with password. • Password for kadmin/admin@TESTDOMAIN.COM: mypassword • kadmin: list_principals • HTTP/ip-10-0-1-244.us-east-2.compute.internal@TESTDOMAIN.COM

Manage the KDC DB principals

kadmin: getprivs
• current privileges: INQUIRE ADD MODIFY DELETE 
• kadmin: cpw ishan • Enter password for principal "ishan@TESTDOMAIN.COM": 
• Re-enter password for principal "ishan@TESTDOMAIN.COM": 
• Password for "ishan@TESTDOMAIN.COM" changed.
Configuration file
client side /etc/krb5.conf
server side /var/kerberos/krb5kdc.conf
chain on 4 steps to access the service via kerberos


in external KDC the shared KDC clusters can access each other hdfs and yarn resources

Authorization EMRFS mapping roles 
When an AWS s3 request is made through EMRFS, each basis for acces is evaluatted in order
EMRFS assumed the corresponding IAM role for the first match--> cluster users and group or s3
prefix as the basis of access. if no baiss for access matches the request,EMRFS uses the cluster EMR role for ec2

How to troubleshoot IAM roles for EMRFS
sudo su -ishan
export HADOOP_ROOT_LOGGER=DEBUG,CONSOLE
hdfs dfs -put text.txt s3://ishanrm-test/test/
it will then later show the debug logs

Why emrfs role mapping without kerberos are not useful?
because HADOOP_USER_NAME =ishan can be assumed

EMRFS role mapping 

EMRFS is a library used in EMR. EMRFS role mapping only works when it is invoked using Hadoop related applications. E.g. If you use "hdfs dfs -put" to write a file to S3, EMRFS will be invoked and use the corresponding role defined in role mapping based on the rules. If you use Spark or Hive to write on S3, EMRFS will be invoked. 
This is because internally both Spark/Hive and HDFS can call the EMRFS jar file when the file/table location starts with s3:// prefix. However, if the file is copied to S3 using "aws s3 cp" or any other AWS S3 CLI command, EMRFS will not be invoked even when the command is run from an EMR cluster, 
because AWS CLI uses Boto3/Botocore internally which does not call EMRFS java class.
[+] https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-fs.html
lease note that if you are running any AWS S3 CLI commands directly after connecting to any EMR node,
 you will not be able to restrict the S3 access through EMRFS role mapping feature.

Intransit encryption concepts
Truststore-- it is the list of keys and certificate of people or servers that you trust.
Keystore-- saves your keys and certificate 

In transit encryption --pem file

zip for the certificate includes:
privatekey.pem mandatory private key
CertificateChain.pem mandatory certicate chain
TrustedCertificates.pem opetion 

with in transit encryption following files will be created and applied
/usr/share/aws/emr/security/conf/trusterstore.jks
/usr/share/aws/emr/security/conf/keystore.kjs

openssl x509 -in certificate.pem -text
Key store has chain of certificate

If in-transit encryption is enabled on an EMR cluster, different network endpoints use different encryption mechanisms. 

For the most in-transit encryption coverage, we recommend that you enable both in-transit encryption and Kerberos. 
If you only enable in-transit encryption, then in-transit encryption will be available only for the network endpoints that support TLS. 
Kerberos is necessary because some open-source framework network endpoints use Simple Authentication and Security Layer (SASL) for in-transit encryption.

The following application-specific encryption features can be enabled using security configurations:
• Hadoop (for more information, see Hadoop in Secure Mode in Apache Hadoop documentation):
• Hadoop MapReduce Encrypted Shuffle uses TLS.
• Secure Hadoop RPC is set to "Privacy" and uses SASL (activated in Amazon EMR when at-rest encryption is
enabled).
• Data encryption on HDFS block data transfer uses AES 256 (activated in Amazon EMR when at-rest encryption is
enabled in the security configuration).
• HBase:
• When Kerberos is enabled, the hbase.rpc.protection property is set to privacy for encrypted communication. For
more information, see Client-side Configuration for Secure Operation in Apache HBase documentation. For more
information about Kerberos with Amazon EMR, see Use Kerberos Authentication.

Presto:
• Internal communication between Presto nodes uses SSL/TLS (Amazon EMR version 5.6.0 and later
only).
• Tez:
• Tez Shuffle Handler uses TLS (tez.runtime.ssl.enable).
• Spark (for more information, see Spark security settings):
• Internal RPC communication between Spark components, such as the block transfer service and the
external shuffle service, is encrypted using the AES-256 cipher in Amazon EMR versions 5.9.0 and
later. In earlier releases, internal RPC communication is encrypted using SASL with DIGEST-MD5 as the
cipher.
• HTTP protocol communication with user interfaces such as Spark History Server and HTTPS-enabled
file servers is encrypted using Spark's SSL configuration. For more information, see SSL
Configuration in Spark documentation.

The EBS ecryption began from EMR 5.24.0 -- EBS encryption option encrypt the ebs root volume and attached stoarage volume
we recommend using EBS encryption

LUKS encryption 
if you choose to use luks encryption for Amazon EBS volume
The lucks encryption applies only to attached stoarage not the root device volume

At rest encryption --cluster provisioning:
on the intance controller, before Bootstrap action we have one system action
/usr/bin/run-system-action VERIFY_DISK_ENCRYPTION

If you don't want anyone to sniff your traffic enable both in-transit and At rest encryption