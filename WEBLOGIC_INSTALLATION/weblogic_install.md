Setup
The following actions should be performed by the "root" user.

Make sure the "/etc/hosts" file contains correct entries for both the "localhost" and real host names.

127.0.0.1      localhost localhost.localdomain localhost4 localhost4.localdomain4
192.168.56.103 ol6.localdomain ol6
Create a new group and user.

groupadd -g 1000 oinstall
useradd -u 1100 -g oinstall oracle
passwd oracle
Create the directories in which the Oracle software will be installed.

mkdir -p /u01/app/oracle/middleware
mkdir -p /u01/app/oracle/config/domains
mkdir -p /u01/app/oracle/config/applications
chown -R oracle:oinstall /u01
chmod -R 775 /u01/
Run the following commands to set your environment and append them to the "/home/oracle/.bash_profile" file.

export MW_HOME=/u01/app/oracle/middleware
export WLS_HOME=$MW_HOME/wlserver
export WL_HOME=$WLS_HOME
# Set to the appropriate JAVA_HOME.
export JAVA_HOME=/u01/app/oracle/jdk1.8.0_77
export PATH=$JAVA_HOME/bin:$PATH
Install JDK
Installing the JDK is simply a matter of extracting it from the tarball.

cd /u01/app/oracle/
tar -xvzf /u01/software/jdk-8u77-linux-x64.tar.gz

# It may just have the following extension.
tar -xvzf /u01/software/jdk-8u77-linux-x64.gz
Create Response File
There are sample response files in the documentation (here), which you will need to adjust to suit your needs. Here is the response file used for WebLogic Server 12c, which should be saved as "/u01/software/wls.rsp".

[ENGINE]
Response File Version=1.0.0.0.0
[GENERIC]
ORACLE_HOME=/u01/app/oracle/middleware
INSTALL_TYPE=WebLogic Server
MYORACLESUPPORT_USERNAME=
MYORACLESUPPORT_PASSWORD=<SECURE VALUE>
DECLINE_SECURITY_UPDATES=true
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
PROXY_HOST=
PROXY_PORT=
PROXY_USER=
PROXY_PWD=<SECURE VALUE>
COLLECTOR_SUPPORTHUB_URL=
For Fusion Middleware Installer you need a response file like the following, saved as "/u01/software/fmw_infr.rsp".

[ENGINE]
Response File Version=1.0.0.0.0
[GENERIC]
ORACLE_HOME=/u01/app/oracle/middleware
INSTALL_TYPE=Fusion Middleware Infrastructure
MYORACLESUPPORT_USERNAME=
MYORACLESUPPORT_PASSWORD=<SECURE VALUE>
DECLINE_SECURITY_UPDATES=true
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
PROXY_HOST=
PROXY_PORT=
PROXY_USER=
PROXY_PWD=<SECURE VALUE>
COLLECTOR_SUPPORTHUB_URL=
You also need to specify an Oracle inventory location. Create a file called "/u01/software/oraInst.loc" with the following contents.

inventory_loc=/u01/app/oraInventory
inst_group=oinstall
WebLogic Silent Installation
The following command shows how to initiate the installation in silent mode.

# WLS
$JAVA_HOME/bin/java -Xmx1024m -jar /u01/software/fmw_12.2.1.0.0_wls.jar -silent -responseFile /u01/software/wls.rsp -invPtrLoc /u01/software/oraInst.loc

# Infrastructure
$JAVA_HOME/bin/java -Xmx1024m -jar /u01/software/fmw_12.2.1.0.0_infrastructure.jar -silent -responseFile /u01/software/fmw_infr.rsp -invPtrLoc /u01/software/oraInst.loc
Once extracted, you can test the WebLogic version using the following command.

. $WLS_HOME/server/bin/setWLSEnv.sh
java weblogic.version
Patching WebLogic Server
Applying patches to WebLogic Server 12c is done using the OPatch utility. Make sure any processes running under this WebLogic installation are stopped before applying a patch.

Download the latest version of OPatch and the WebLogic updates from Oracle Support and put them into the "/u01/software" directory with the other software. For example.

OPatch : p6880880_132000_Generic.zip *** See Note Below
Updates : p22331568_122100_Generic.zip
 At the time of writing, the latest version of OPatch (OUI NextGen 13.2) is older than the version of OPatch that ships with WebLogic Server 12.2.1.

Assuming you are not using WebLogic Server 12.2.1, unzip the OPatch utility and add it to your path.

cd /u01/software
unzip p6880880_132000_Generic.zip
export PATH=/u01/software/OPatch:$PATH
If you are using WebLogic Server 12.2.1, set the existing OPatch location in your path.

export PATH=$MW_HOME/OPatch:$PATH
Unzip the patch and change to the resulting directory, then apply the patch.

unzip -d PATCH_TOP p22331568_122100_Generic.zip
cd PATCH_TOP/22331568
export ORACLE_HOME=$MW_HOME
opatch apply -silent
Answer any prompts and take appropriate action when required.

When the patch is complete, check the WebLogic version using the following command. Depending on the scale of the patch, the version may not have changed.

. $WLS_HOME/server/bin/setWLSEnv.sh
java weblogic.version
For more information see:

Oracle WebLogic Server (WLS) 12cR2 (12.2.1) Installation on Oracle Linux 6 and 7
Oracle Fusion Middleware Installing with the Oracle Universal Installer : Using the Oracle Universal Installer in Silent Mode
Hope this helps. Regards Tim...