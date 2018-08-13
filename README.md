# SAS Viya Deployment Assistant for Windows (sas-wvda.ps1)

## Purpose
To ease the installation process of SAS Viya 3.4 on Microsoft Windows Server by:
* Validating pre-requisites are meet
* Validating / applying SAS Recommended tuning
* Installing SAS public certificates to facilitate execution of signed scripts
* Validating / configuring user and service principals required by the software.
* Validate encrypted passwords configured by SAS Admin
* facilitate the communication process between the SAS Admin and the domain / security admin

## Audience
The primary users of sas-wvda are SAS Administrators.  

## Usage Pre-reqs
* Java 8 must be Installed
* the JAVA_HOME environment variable must be correctly set
* Powershell 5.1 must be Installed
* The user must be a local administrator of the host
* This script sas-wvda.ps1 should be executed only on a server that is intended to run SAS Viya.
* If the user is also a domain admin then additional functionality is available

## Output classification
sas-wvda produces 5 types of output messages:
* INFO: These messages are informational messages such as version information and feedback that a requirement has been verified
* WARNING: These messages indicate a requirement has not yet been met.  WARNING messages must be resolved before SAS Viya can be successfully installed.
* NOTE: These messages indicate that the deployment assistant has made a change to your environment.  This change may be creating an account, changing a tuning parameter, or adding an account to a local group.  These messages will be of interest to you from a system administrative perspective.
* ERROR: These messages indicate that a problem has occurred during processing.
* unclassified messages are generated by some commands invoked by the deployment assistant.

## Documentation
The most up-to-date documentation can be obtained from the script itself via the [Get-Help command](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-help?view=powershell-5.1)
for example:

`Get-Help C:\path\to\sas-wvda.ps1
Get-Help C:\path\to\sas-wvda.ps1 -examples
Get-Help C:\path\to\sas-wvda.ps1 -detailed
Get-Help C:\path\to\sas-wvda.ps1 -full`

Note the -Verbose argument can be used to obtain detailed information in the event of unexpected results.

## Usage overview
To run this script the SAS administrator must be an administrator on the local host.  If the SAS Administrator has domain
admin privileges then the script can completely prepare a host for deployment of SAS Viya 3.4 on Windows.  This documentation will walk you through the process of preparing a server for SAS Viya installation.  There are two scenarios presented:
* User has local administrator privileges but not domain administrator privileges
* User has both local administrator and domain administrator privileges

### Pre-installation walkthrough where user is a local Admin but not a Domain Admin
If the user is a local admin but does not have domain administrative privileges the process of preparing for the installation of SAS Viya will require assistance from someone who has domain administrative privileges.  An overview of the steps includes:
* local admin runs sas-wvda to assesses the readiness of the server and remediates any issues identified
* local admin runs sas-wvda to prepare a script for the domain administrator to run.  The script will automate all actions required on the part of the domain administrator to configure all domain level entities and permissions required for SAS Viya to function.
* The local admin provides the script to their domain administrator
* The domain administrator executes the script.  At the end of the script a zip file is prepared which must be given to the local admin to continue configuration of the server that is to host SAS Viya.
* The local admin uses the content of the zip file to continue preparations for the installation of SAS Viya
* Once all WARNINGs and ERRORs are resolved the local administrator can install SAS Viya.

 The first step is to assess the basic install readiness of the host:

`c:\path\to\sas-wvda.ps1 -validate host`

In this example many prerequisites are not met.  None of the account entities exist.
There are system tuning requirements that are not met.:
```
PS C:\viya\sas-wvda> .\sas-wvda.ps1 -validate host

INFO: sas-wvda version 1.1.01

INFO: Executing on host: WIN2016D
WARNING: JAVA_HOME is not set!
INFO: The version of the .NET Framework is 4.6 or higher
INFO: Running in a 64-bit environment: OK
INFO: Running PowerShell 5.1 or higher: OK
INFO: Running on Windows Server: OK
INFO: Server is part of a domain: OK
INFO: Running on Microsoft Windows Server 2016 Standard: OK
WARNING: The 64-bit Microsoft Visual C++ 2013 Redistributable Package must be installed on this host prior to installing SAS Viya.
         Download and execute the appropriate vcredist_x64.exe from
         https://support.microsoft.com/en-us/help/3179560/update-for-visual-c-2013-and-visual-c-redistributable-package.
WARNING: The 64-bit Microsoft Visual C++ 2015 Redistributable Package must be installed on this host prior to installing SAS Viya.
         Download and execute vc_redist.x64.exe
         from https://www.microsoft.com/en-us/download/details.aspx?id=48145
         Installing the 64-bit Microsoft Visual C++ 2017 Redistributable Package will also satisfy this requirement.
         The 2017 package can be obtained from https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads
WARNING: Host Account for WIN2016D is not trusted for delegation!
WARNING: SPN sascas/win2016d.ad.perf.sas.com is not defined!
WARNING: SPN HTTP/win2016d.ad.perf.sas.com is not defined!
WARNING: Keytab Path not specified.  Keytab can not be validated.
WARNING: Keytab not found at  - Keytab will not be validated.
WARNING: The Postgres service account was not found
WARNING: SharedSection Tuning does not match requested value!
         The third parameter of the Windows Subsystem Shared Section triplet must be at least 20480!
         The current Value is 768.  To Remediate run script with the -remediate switch or open regedit,
         find HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\SubSystems\Windows and change
         SharedSection=1024,20480,768 to read SharedSection=1024,20480,20480
WARNING: TcpTimedWaitDelay value is not set!  SAS recommends this value be set to 30
WARNING: Win32PrioritySeparation is set to 1.  SAS recommends this value be set to 36.
INFO: TCP ephemeral port range start value (32768) 32768 or less: OK
INFO: TCP ephemeral port quantity (32767) 32767 or greater: OK
WARNING: SAS Public Certs are not installed!
```
This README will not provided detailed instructions for this but the local administrator must visit the URLs provided in the sas-wvda output to obtain and install the C++ redistributable packages.

Set JAVA_HOME eg:
`$env:JAVA_HOME="C:\Program Files\Java\jre1.8.0_172\"`

Run the deployment assistant to remediate issues on the local host and create a script to be provided to the domain administrator
`c:\path\to\sas-wvda.ps1 -validate host -Remediate -CreateADEntities -CreateKeytab -PostgresAcct "postgres" -casAcct "cas"`

```
PS C:\viya\sas-wvda> .\sas-wvda.ps1 -validate host -Remediate -CreateADEntities -CreateKeytab -PostgresAcct "postgres" -casAcct "cas"

INFO: sas-wvda version 1.1.01

INFO: Executing on host: WIN2016D
INFO: JAVA_HOME (C:\Program Files\Java\jre1.8.0_172\) points to an installation of Java 8: OK
INFO: 64-bit version of Java 8 found in JAVA_HOME: OK
INFO: The version of the .NET Framework is 4.6 or higher
INFO: Running in a 64-bit environment: OK
INFO: Running PowerShell 5.1 or higher: OK
WARNING: The current account is not a Domain Admin yet -createADEntities has been specified.
         Will not attempt to modify domain entities.  A script will be created instead.
INFO: Running on Windows Server: OK
INFO: Server is part of a domain: OK
INFO: Running on Microsoft Windows Server 2016 Standard: OK
INFO: Microsoft Visual C++ 2017 Redistributable (x64) - 14.14.26429 is installed: OK
INFO: Microsoft Visual C++ 2013 Redistributable (x64) - 12.0.40660 is installed: OK
WARNING: Keytab Path not specified.  Keytab can not be validated.
WARNING: Keytab not found at  - Keytab will not be validated.
NOTE: The Postgres Service Account (WIN2016D\postgres) was successfully created with password: 2nY|*hn8.#E4-E{=5!2U
NOTE: Added WIN2016D\postgres to the Users group
NOTE: Granted WIN2016D\postgres the seServiceLogonRight right
NOTE: Updated Windows subsystem SharedSection tuning from SharedSection=1024,20480,768 to SharedSection=1024,20480,20480
NOTE: Adding HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\TcpTimedWaitDelay with value: 30
NOTE: Updating Win32PrioritySeparation from: 1 to: 36
INFO: TCP ephemeral port range start value (32768) 32768 or less: OK
INFO: TCP ephemeral port quantity (32767) 32767 or greater: OK
INFO: SAS Public Cert update: OK
      CERTUTIL output:
      TrustedPublisher "Trusted Publishers"
      Certificate "SAS Institute Inc." added to store.
      Certificate "SAS Institute, Inc." added to store.
      CertUtil: -addstore command completed successfully.
Provide SASViyaADEntitySetup.ps1 to your Active Directory administrator.  When the script has successfully completed you will be provided:
 - The name and password of the CAS account
 - A keytab for the HTTP service principal
You must have all of these artifacts prior to deployment of SAS Viya 3.4.
NOTE: System settings have been updated which require a restart to become effective
      This system must be restarted prior to installing SAS Viya!
```
**Take care to follow the instruction at the end and reboot if instructed to do so otherwise SAS Viya will not successfully install and start**

Provide the SASViyaADEntitySetup.ps1 script to your domain administrator.

Once the domain administrator has executed the script and provided the ViyaAdminInfo.zip file you will need to extract the contents.  Place the keytab file in the location of your choosing.  Use the content of the ViyaAdminREADME.txt file to understand the actions taken on your behalf.  You will also need the name of the CAS service account as well as the password to prepare for the SAS Viya deployment.

Sample content from a ViyaAdminREADME.txt file:
```
Actions Completed by domain admin helper script:

The CAS Service Account (ADPERF\cas) was successfully created with password: %1N;^yt-!o}g+kztpg-q
The HTTP service account (ADPERF\WIN2016D-HTT) was created successfully.
The HTTP keytab (http-WIN2016D.keytab) was successfully created.
      Targeting domain controller: win2012.ad.perf.sas.com
      Password successfully set!
```
Once the keytab is copied to the desired location and the passwords provided for the postgres and cas accounts are used to create the encrypted user files it is time to run the deployment assistant again to complete configuration on the local host and to ensure all the local rights and group memberships are configured.

`c:\path\to\sas-wvda.ps1 C:\path\to\http-host.keytab -validate all -Remediate -PostgresAcct "postgres" -DeployDir C:\path\to\deploydir\powershell-deployment`
```
PS C:\viya\powershell-deployment> C:\viya\sas-wvda\sas-wvda.ps1 C:\viya\http-WIN2016D.keytab -validate all -DeployDir C:\viya\powershell-deployment\ -remediate

INFO: sas-wvda version 1.1.01

INFO: Executing on host: WIN2016D
INFO: JAVA_HOME (C:\Program Files\Java\jre1.8.0_172\) points to an installation of Java 8: OK
INFO: 64-bit version of Java 8 found in JAVA_HOME: OK
INFO: The version of the .NET Framework is 4.6 or higher
INFO: Running in a 64-bit environment: OK
INFO: Running PowerShell 5.1 or higher: OK
INFO: Running on Windows Server: OK
INFO: Server is part of a domain: OK
INFO: Running on Microsoft Windows Server 2016 Standard: OK
INFO: Microsoft Visual C++ 2017 Redistributable (x64) - 14.14.26429 is installed: OK
INFO: Microsoft Visual C++ 2013 Redistributable (x64) - 12.0.40660 is installed: OK
INFO: Validate Postgres User Credentials: OK
INFO: Validate CAS User Credentials: OK
INFO: Host Account for WIN2016D trusted for delegation: OK
INFO: sascas/win2016d.ad.perf.sas.com is defined: OK
INFO: sascas SPN Account Name = ADPERF\svc-sas-WIN2016D-CAS
INFO: Stored CAS Username matches SPN User: OK
INFO: ADPERF\svc-sas-WIN2016D-CAS is trusted for delegation: OK
NOTE: Granted ADPERF\svc-sas-WIN2016D-CAS the SeAssignPrimaryTokenPrivilege Right
NOTE: Granted ADPERF\svc-sas-WIN2016D-CAS the seServiceLogonRight right
NOTE: Added ADPERF\svc-sas-WIN2016D-CAS to the local Administrators group
INFO: HTTP/win2016d.ad.perf.sas.com is defined: OK
INFO: HTTP SPN Account Name = ADPERF\svc-sas-WIN2016D-HTT
INFO: KINIT using keytab: OK
      KINIT output:
      New ticket is stored in cache file C:\Users\kentest\krb5cc_kentest
      Picked up _JAVA_OPTIONS: -Dsun.security.krb5.debug=false -Djava.security.krb5.conf=C:\viya\sas-wvda\krb5.ini
INFO: The postgres service account exists: OK
INFO: The postgres service account is a member of the local Users group: OK
INFO: postgres has Log on as a Service right: OK
INFO: Windows subsystem SharedSection tuning(SharedSection=1024,20480,20480) meets minimum: OK
INFO: TcpTimedWaitDelay is set to 30 : OK
INFO: Win32PrioritySeparation is set to 36 : OK
INFO: TCP ephemeral port range start value (32768) 32768 or less: OK
INFO: TCP ephemeral port quantity (32767) 32767 or greater: OK
INFO: SAS public code signing certs are installed: OK
```

Because there are no WARNINGs SAS Viya can be successfully deployed on this host.  For specific instructions on these steps refer to the __SAS Viya on Windows: Deployment Guide__ for your version of SAS Viya software.

### Pre-installation walkthrough where user is Domain Admin
If the user is both a local admin and domain admin the first step is to assess the basic install readiness of the host:

`c:\path\to\sas-wvda.ps1 -validate host`

In this example many prerequisites are not met.  None of the account entities exist.
There are system tuning requirements that are not met.:
```
PS C:\viya\sas-wvda> .\sas-wvda -validate host

INFO: sas-wvda version 0.1.33

INFO: Executing on host: WIN2016D
INFO: JAVA_HOME (C:\program files\java\jre1.8.0_172\) points to an installation of Java 8: OK
INFO: 64-bit version of Java 8 found in JAVA_HOME: OK
INFO: The version of the .NET Framework is 4.6 or higher
INFO: Running in a 64-bit environment: OK
INFO: Running PowerShell 5.1 or higher: OK
INFO: Currently running as a Domain Admin...
INFO: Running on Windows Server: OK
INFO: Server is part of a domain: OK
INFO: Running on Microsoft Windows Server 2016 Standard: OK
WARNING: The 64-bit Microsoft Visual C++ 2013 Redistributable Package must be installed on this host prior to installing SAS Viya.
         Download and execute the appropriate vcredist_x64.exe from
         https://support.microsoft.com/en-us/help/3179560/update-for-visual-c-2013-and-visual-c-redistributable-package.
WARNING: The 64-bit Microsoft Visual C++ 2015 Redistributable Package must be installed on this host prior to installing SAS Viya.
         Download and execute vc_redist.x64.exe
         from https://www.microsoft.com/en-us/download/details.aspx?id=48145
         Installing the 64-bit Microsoft Visual C++ 2017 Redistributable Package will also satisfy this requirement.
         The 2017 package can be obtained from https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads
WARNING: Host Account for WIN2016D is not trusted for delegation!
WARNING: SPN sascas/win2016d.ad.perf.sas.com is not defined!
WARNING: SPN HTTP/win2016d.ad.perf.sas.com is not defined!
WARNING: Keytab Path not specified.  Keytab can not be validated.
WARNING: Keytab not found at  - Keytab will not be validated.
WARNING: The Postgres service account was not found
WARNING: SharedSection Tuning does not match requested value!
         The third parameter of the Windows Subsystem Shared Section triplet must be at least 20480!
         The current Value is 768.  To Remediate run script with the -remediate switch or open regedit,
         find HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\SubSystems\Windows and change
         SharedSection=1024,20480,768 to read SharedSection=1024,20480,20480
WARNING: TcpTimedWaitDelay value is not set!  SAS recommends this value be set to 30
WARNING: Win32PrioritySeparation is set to 1.  SAS recommends this value be set to 36.
INFO: TCP ephemeral port range start value (32768) 32768 or less: OK
INFO: TCP ephemeral port quantity (32767) 32767 or greater: OK
WARNING: SAS Public Certs installed are not installed!
```
After installing the required redistributable packages from Microsoft run the script again.  In this walkthrough we have Domain Admin privileges so we are able to create entities in Active Directory as well as on the local host.  To create all accounts with specified names, grant permissions, and create a keytab invoke the deployment assistant as follows:

`c:\path\to\sas-wvda.ps1 c:\path\to\ViyaHTTP.keytab -Validate host -SvcAcctPrefix "" -PostgresAcct "postgres" -CasAcct "cas" -CreateADEntities -CreateKeytab -Remediate`
```
PS C:\viya\sas-wvda> .\sas-wvda.ps1 C:\viya\http-WIN2016D.keytab -Validate host -CreateADEntities -CreateKeytab -Remediate -PostgresAcct "postgres" -casAcct "cas" -svcAcctPrefix ""

INFO: sas-wvda version 0.1.33

INFO: Executing on host: WIN2016D
INFO: JAVA_HOME (C:\program files\java\jre1.8.0_172\) points to an installation of Java 8: OK
INFO: 64-bit version of Java 8 found in JAVA_HOME: OK
INFO: The version of the .NET Framework is 4.6 or higher
INFO: Running in a 64-bit environment: OK
INFO: Running PowerShell 5.1 or higher: OK
INFO: Currently running as a Domain Admin...
INFO: Running on Windows Server: OK
INFO: Server is part of a domain: OK
INFO: Running on Microsoft Windows Server 2016 Standard: OK
INFO: Microsoft Visual C++ 2017 Redistributable (x64) - 14.14.26429 is installed: OK
INFO: Microsoft Visual C++ 2013 Redistributable (x64) - 12.0.40660 is installed: OK
NOTE: The CAS Service Account (ADPERF\cas) was successfully created with password: hIe#u!fO&(Z/(tT=I@Wr
INFO: The HTTP service account (ADPERF\WIN2016D-HTTP) was created successfully.
INFO: sleeping for 5 seconds to allow replication.
Successfully mapped HTTP/win2016d.ad.perf.sas.com to WIN2016D-HTTP.
Key created.
Key created.
Key created.
Key created.
Key created.
Output keytab to C:\viya\http-WIN2016D.keytab:
Keytab version: 0x502
keysize 72 HTTP/win2016d.ad.perf.sas.com@AD.PERF.SAS.COM ptype 1 (KRB5_NT_PRINCIPAL) vno 0 etype 0x1 (DES-CBC-CRC) keylength 8 (0xc183d6b09873a2cb)
keysize 72 HTTP/win2016d.ad.perf.sas.com@AD.PERF.SAS.COM ptype 1 (KRB5_NT_PRINCIPAL) vno 0 etype 0x3 (DES-CBC-MD5) keylength 8 (0xc183d6b09873a2cb)
keysize 80 HTTP/win2016d.ad.perf.sas.com@AD.PERF.SAS.COM ptype 1 (KRB5_NT_PRINCIPAL) vno 0 etype 0x17 (RC4-HMAC) keylength 16 (0xe099c6ce300eb8a07c6f245b5a9568bf)
keysize 96 HTTP/win2016d.ad.perf.sas.com@AD.PERF.SAS.COM ptype 1 (KRB5_NT_PRINCIPAL) vno 0 etype 0x12 (AES256-SHA1) keylength 32 (0xab7077ad91bf0971ed5ddae0b031d1d4522757dcdd29398beae5e54a652903c1)
keysize 80 HTTP/win2016d.ad.perf.sas.com@AD.PERF.SAS.COM ptype 1 (KRB5_NT_PRINCIPAL) vno 0 etype 0x11 (AES128-SHA1) keylength 16 (0x30c03541ca0a333e68dbacd31e1d6a3f)
NOTE: The HTTP keytab (C:\viya\http-WIN2016D.keytab) was successfully created.
      Targeting domain controller: win2012.ad.perf.sas.com
      Using legacy password setting method
INFO: KINIT using keytab: OK
      KINIT output:
      New ticket is stored in cache file C:\Users\kegaha\krb5cc_kegaha
      Picked up _JAVA_OPTIONS: -Dsun.security.krb5.debug=false -Djava.security.krb5.conf=C:\viya\sas-wvda\krb5.ini
NOTE: The Postgres Service Account (WIN2016D\postgres) was successfully created with password: oc*Zp9:>!Pfd(Kk)6}=x
NOTE: Added WIN2016D\postgres to the Users group
NOTE: Granted WIN2016D\postgres the seServiceLogonRight right
NOTE: Updated Windows subsystem SharedSection tuning from SharedSection=1024,20480,768 to SharedSection=1024,20480,20480
NOTE: Adding HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\TcpTimedWaitDelay with value: 30
NOTE: Updating Win32PrioritySeparation from: 1 to: 36
INFO: TCP ephemeral port range start value (32768) 32768 or less: OK
INFO: TCP ephemeral port quantity (32767) 32767 or greater: OK
INFO: SAS Public Cert update: OK
      CERTUTIL output:
      TrustedPublisher "Trusted Publishers"
      Certificate "SAS Institute Inc." added to store.
      Certificate "SAS Institute, Inc." added to store.
      CertUtil: -addstore command completed successfully.
NOTE: System settings have been updated which require a restart to become effective
      This system must be restarted prior to installing SAS Viya!
```
**Take care to follow the instruction at the end and reboot if instructed to do so otherwise SAS Viya will not successfully install and start**

Here we see that accounts have been created for CAS, PostrgreSQL, and the passwords for those accounts have been displayed in the tool output.  Note that a service account has also been created for the HTTP service on this host.  If there had been a desire to specify the passwords for these accounts the -CASPassword and -PostgresPassword arguments could have been used to specify those on the command invocation.

Now execute the command one final time to ensure all rights have been assigned to the accounts created above.  This time we do not need the -CreateADEntities and -CreateKeytab arguments

`c:\path\to\sas-wvda.ps1 c:\path\to\ViyaHTTP.keytab -Validate host -SvcAcctPrefix "" -PostgresAcct "postgres" -CasAcct "cas" -Remediate`
```
PS C:\viya\sas-wvda> .\sas-wvda.ps1 C:\viya\http-WIN2016D.keytab -Validate host -Remediate -PostgresAcct "postgres" -casAcct "cas" -svcAcctPrefix ""

INFO: sas-wvda version 0.1.33

INFO: Executing on host: WIN2016D
INFO: JAVA_HOME (C:\program files\java\jre1.8.0_172\) points to an installation of Java 8: OK
INFO: 64-bit version of Java 8 found in JAVA_HOME: OK
INFO: The version of the .NET Framework is 4.6 or higher
INFO: Running in a 64-bit environment: OK
INFO: Running PowerShell 5.1 or higher: OK
INFO: Currently running as a Domain Admin...
INFO: Running on Windows Server: OK
INFO: Server is part of a domain: OK
INFO: Running on Microsoft Windows Server 2016 Standard: OK
INFO: Microsoft Visual C++ 2017 Redistributable (x64) - 14.14.26429 is installed: OK
INFO: Microsoft Visual C++ 2013 Redistributable (x64) - 12.0.40660 is installed: OK
INFO: Host Account for WIN2016D trusted for delegation: OK
INFO: sascas/win2016d.ad.perf.sas.com is defined: OK
INFO: sascas SPN Account Name = ADPERF\cas
INFO: ADPERF\cas is trusted for delegation: OK
NOTE: Granted ADPERF\cas the SeAssignPrimaryTokenPrivilege Right
NOTE: Granted ADPERF\cas the seServiceLogonRight right
NOTE: Added ADPERF\cas to the local Administrators group
INFO: HTTP/win2016d.ad.perf.sas.com is defined: OK
INFO: HTTP SPN Account Name = ADPERF\WIN2016D-HTTP
INFO: KINIT using keytab: OK
      KINIT output:
      New ticket is stored in cache file C:\Users\kegaha\krb5cc_kegaha
      Picked up _JAVA_OPTIONS: -Dsun.security.krb5.debug=false -Djava.security.krb5.conf=C:\viya\sas-wvda\krb5.ini
INFO: The postgres service account exists: OK
INFO: The postgres service account is a member of the local Users group: OK
INFO: postgres has Log on as a Service right: OK
INFO: Windows subsystem SharedSection tuning(SharedSection=1024,20480,20480) meets minimum: OK
INFO: TcpTimedWaitDelay is set to 30 : OK
INFO: Win32PrioritySeparation is set to 36 : OK
INFO: TCP ephemeral port range start value (32768) 32768 or less: OK
INFO: TCP ephemeral port quantity (32767) 32767 or greater: OK
INFO: SAS public code signing certs are installed: OK
```

Once all host validations have passed you are ready to generate your deployment orchestration scripts.  For specific instructions on these steps refer to the __SAS Viya on Windows: Deployment Guide__ for your version of SAS Viya software.  Prior to invoking the actual deployment command encrypted passwords and other configuration requirements can be verified by invoking the deployment assistant again:

`C:\path\to\sas-wvda.ps1 c:\path\to\ViyaHTTP.keytab -validate all -DeployDir c:\path\to\sas\viya\powershell_script`

```
PS C:\viya\powershell-deployment> ..\sas-wvda\sas-wvda.ps1 C:\viya\http-WIN2016D.keytab -Validate all -DeployDir C:\viya\powershell-deployment\

INFO: sas-wvda version 0.1.33

INFO: Executing on host: WIN2016D
INFO: JAVA_HOME (C:\program files\java\jre1.8.0_172\) points to an installation of Java 8: OK
INFO: 64-bit version of Java 8 found in JAVA_HOME: OK
INFO: The version of the .NET Framework is 4.6 or higher
INFO: Running in a 64-bit environment: OK
INFO: Running PowerShell 5.1 or higher: OK
INFO: Currently running as a Domain Admin...
INFO: Running on Windows Server: OK
INFO: Server is part of a domain: OK
INFO: Running on Microsoft Windows Server 2016 Standard: OK
INFO: Microsoft Visual C++ 2017 Redistributable (x64) - 14.14.26429 is installed: OK
INFO: Microsoft Visual C++ 2013 Redistributable (x64) - 12.0.40660 is installed: OK
INFO: Validate Postgres User Credentials: OK
INFO: Validate CAS User Credentials: OK
INFO: Host Account for WIN2016D trusted for delegation: OK
INFO: sascas/win2016d.ad.perf.sas.com is defined: OK
INFO: sascas SPN Account Name = ADPERF\cas
INFO: Stored CAS Username matches SPN User: OK
INFO: ADPERF\cas is trusted for delegation: OK
INFO: cas has Replace Process Level Token: OK
INFO: cas has Log on as a Service right: OK
INFO: ADPERF\cas is a member of the local Administrators group: OK
INFO: HTTP/win2016d.ad.perf.sas.com is defined: OK
INFO: HTTP SPN Account Name = ADPERF\WIN2016D-HTTP
INFO: KINIT using keytab: OK
      KINIT output:
      New ticket is stored in cache file C:\Users\kegaha\krb5cc_kegaha
      Picked up _JAVA_OPTIONS: -Dsun.security.krb5.debug=false -Djava.security.krb5.conf=C:\viya\sas-wvda\krb5.ini
INFO: The postgres service account exists: OK
INFO: The postgres service account is a member of the local Users group: OK
INFO: postgres has Log on as a Service right: OK
INFO: Windows subsystem SharedSection tuning(SharedSection=1024,20480,20480) meets minimum: OK
INFO: TcpTimedWaitDelay is set to 30 : OK
INFO: Win32PrioritySeparation is set to 36 : OK
INFO: TCP ephemeral port range start value (32768) 32768 or less: OK
INFO: TCP ephemeral port quantity (32767) 32767 or greater: OK
INFO: SAS public code signing certs are installed: OK
```
In this scenario there are no WARNING or ERROR messages.  The host is now prepared to host a SAS Viya deployment.  For specific instructions on these steps refer to the __SAS Viya on Windows: Deployment Guide__ for your version of SAS Viya software.

## Behavior
If a SAS Administrator has both local and domain admin privileges on the server and network where SAS is to be installed the script can perform the following actions from one invocation:

`c:\path\to\sas-wvda.ps1 c:\path\to\ViyaHTTP.keytab -validate host -createADEntities -remediate -createKeytab`

will:
* Validate the script is executing in a 64-bit environment
* Validate the version of PowerShell meets or exceeds the minimum requirement
* Validate the installation is running on Windows Server
* Validate that the version of Windows Server meets or exceeds the minimum requirement
* Validate that the 64-bit Microsoft Visual C++ 2013 Redistributable is installed
* Validate that the 64-bit Microsoft Visual C++ 2015 Redistributable is installed
* Ensure required modules are installed
* Validate that the server's computer account is trusted for delegation / remediate if not
* Validate that the casspn/<hostname> Service Principal Name (SPN) is defined
	* If the SPN is not defined in Active Directory (AD) searches for an account that matches the default name
	* If the default account exists ensure all required permissions are configured and associate the SPN
	* If the default account does not exist - Create it with required attributes - assigning a random password and associate the SPN with the account - report the new password to the user
	* Ensure the domain account:
		* Is in the local Administrators group
		* Has the "Log on as a service" right
		* Has the "Replace a process level token" right
* Validate that the HTTP/<hostname> Service Principal Name (SPN) is defined
	* If the SPN is not defined in Active Directory (AD) searches for an account that matches the default name
	* If the default account exists ensure all required permissions are configured and associate the SPN
	* If the default account does not exist - Create it with required attributes - assigning a random password and associate the SPN with the account
* Create a keytab for the HTTP/<hostname> SPN named c:\path\to\ViyaHTTP.keytab
* Validate the keytab
* Validate the local postgres service account
	* Exists
	* Is a member of the local Users Group
	* Has the "Log on as a service" right
* Validate / Remediate SharedSection Tuning
* Validate / Remediate TcpTimedWaitDelay value
* Validate / Remediate Win32PrioritySeparation value
* Validate / Remediate the TCP Dynamic Port range configuration meets SAS recommendations
* Validate / Remediate the presence of SAS Public certificates used to validate signed scripts during install and execution of SAS Viya.

If a SAS Administrator has local admin privileges but not domain admin privileges then a command such as:

`c:\path\to\sas-wvda.ps1 c:\sas\ViyaHTTP.keytab -validate host -createADEntities -cmdFileOnly -remediate -createKeytab`

will:
* Validate the script is executing in a 64-bit environment
* Validate the version of PowerShell meets or exceeds the minimum requirement
* Validate the installation is running on Windows Server
* Validate that the version of Windows Server meets or exceeds the minimum requirement
* Validate that the 64-bit Microsoft Visual C++ 2013 Redistributable is installed
* Validate that the 64-bit Microsoft Visual C++ 2015 Redistributable is installed
* Ensure required modules are installed
* Create a new domain-admin-helper PowerShell script that contains commands to remediate any of the following Validations that fail:
	* Validate that the server's computer account is trusted for delegation / remediate if not
	* Validate that the sascas/<hostname> Service Principal Name (SPN) is defined
		* If the SPN is not defined in Active Directory (AD) searches for an account that matches the default name
		* If the default account exists ensure all required permissions are configured and associate the SPN
		* If the default account does not exist - Create it with required attributes - assigning a random password and associate the SPN with the account - report the new password to the user
		* Ensure the domain account:
			* Is in the local Administrators group
			* Has the "Log on as a service" right
			* Has the "Replace a process level token" right
	* Validate that the HTTP/<hostname> Service Principal Name (SPN) is defined
		* If the SPN is not defined in Active Directory (AD) searches for an account that matches the default name
		* If the default account exists ensure all required permissions are configured and associate the SPN
		* If the default account does not exist - Create it with required attributes - assigning a random password and associate the SPN with the account
	* Create a keytab for the HTTP/<hostname> SPN named c:\path\to\ViyaHTTP.keytab
* Validate the local postgres service account
	* Create if doesn't exist
	* If not a member of the local Users Group - add to local Users group
	* If does not have the "Log on as a service" right - grant the right
* Validate / Remediate SharedSection Tuning
* Validate / Remediate TcpTimedWaitDelay value
* Validate / Remediate Win32PrioritySeparation value
* Validate / Remediate the TCP Dynamic Port range configuration meets SAS recommendations
* Validate / Remediate the presence of SAS Public certificates used to validate signed scripts during install and execution of SAS Viya.

The default location for the generated script is the current working directory.  The default name for the script is SASViyaADEntitySetup.ps1
The path and name of the script can be changed by using the -cmdFilePath argument followed by the path and filename desired.

The script can be used immediately prior to the deployment of SAS Viya in validate and report only mode by:

`c:\path\to\sas-wvda.ps1 c:\sas\ViyaHTTP.keytab -deployDir c:\path\to\powershell-deployment`

If -deployDir is not specified then the current working directory is assumed to contain the encrypted password xml files.  In addition to all the validation listed above this invocation will also perform the following validations:
* The postgresUser.xml file contains a valid username and password combination
* The casUser.xml file contains a valid username and password combination
* The username in the casUser.xml file matches the account name associated with the sascas/<hostname> SPN

## Planned enhancements
* After release: use the zip file created by the domain-admin-helper PowerShell script to create and validate the required encrypted credential xml files
* After release: handle the backup service account as the CAS account is currently handled
* After release: Provide the ability to periodically update passwords on the domain accounts and refresh the HTTP SPN's keytab
* After release: perform common troubleshooting for other configuration items that can go wrong such as invalid configuration of the Kerberos properties via SAS Environment Manager