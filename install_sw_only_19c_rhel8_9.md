# Oracle 19c Software Installation Guide (RHEL 8/9)

## Prerequisites

### Configure Swappiness

Lower swappiness to 10 to prevent issues with swapping.

Switch to root user:
```bash
sudo su -
```

Open the sysctl configuration file:
```bash
vi /etc/sysctl.conf
```

Add/modify the following line to set the swappiness value:
```bash
# changed swappiness to reduce swapping
vm.swappiness = 10
```

Apply the changes:
```bash
sysctl -p
```

### Install Oracle Preinstall Package

Install the preinstall package and enable crontab for the oracle user.

Switch to root user:
```bash
sudo su -
```

Enable cron for oracle user:
```bash
echo "oracle" >> /etc/cron.allow 
```

Install the preinstall package (local installation):
```bash
dnf -y localinstall oracle-database-preinstall-19c-1.0-1.<OS_VERSION>.x86_64.rpm
```

Or, if you have access to the internet (for RHEL 9):
```bash
wget -P /tmp/ https://yum.oracle.com/repo/OracleLinux/OL9/appstream/x86_64/getPackage/oracle-database-preinstall-19c-1.0-1.el9.x86_64.rpm
```

```bash
dnf install -y --cacheonly oracle-database-preinstall-19c-1.0-1.el9.x86_64.rpm
```

#### Troubleshooting

If you encounter this error:
```
Failed:
  chkconfig-1.24-1.el9.x86_64                                                                                       
Error: Transaction failed
```

Run these steps:
```bash
mv /etc/init.d/dsmcad /etc/rc.d/init.d/ ; rm -rf /etc/init.d ; ln -s /etc/rc.d/init.d/ /etc/init.d ; dnf install -y chkconfig
```

### Install All Required Packages

Install all needed packages and check that the environment is compliant with Oracle standards.

Switch to root user:
```bash
sudo su -
```

Run the comprehensive installation and validation script:
```bash
bash -c 'echo -e "\033[1;33m=== Oracle 19c Prerequisites Installation and Validation Starting ===\033[0m"; if [ "$(id -u)" -ne 0 ]; then echo -e "\033[1;31m=== ERROR:   This script must be run as root ===\033[0m"; exit 1; fi; if [ -f /etc/os-release ]; then . /etc/os-release; OS_NAME="${NAME}"; OS_VERSION="${VERSION_ID}"; OS_MAJOR_VERSION="${VERSION_ID%%.*}"; echo -e "\033[1;36m=== Detected OS:   ${OS_NAME} ${OS_VERSION} (Major: ${OS_MAJOR_VERSION}) ===\033[0m"; else echo -e "\033[1;31m=== ERROR:  Cannot determine OS version ===\033[0m"; exit 1; fi; if [[ !  "${OS_NAME}" =~ (Red Hat Enterprise Linux|Oracle Linux) ]]; then echo -e "\033[1;31m=== ERROR:  Unsupported OS:   ${OS_NAME}.   This script supports RHEL/Oracle Linux 8 or 9 only ===\033[0m"; exit 1; fi; if [[ "${OS_MAJOR_VERSION}" != "8" && "${OS_MAJOR_VERSION}" != "9" ]]; then echo -e "\033[1;31m=== ERROR:  Unsupported OS version:  ${OS_VERSION}. This script supports RHEL/Oracle Linux 8 or 9 only ===\033[0m"; exit 1; fi; echo -e "\033[1;36m=== Clearing DNF cache ===\033[0m"; if ! dnf clean all -q; then echo -e "\033[1;31m=== ERROR:  Failed to clear DNF cache ===\033[0m"; exit 1; fi; echo -e "\033[1;32m=== DNF cache cleared successfully ===\033[0m"; if [ "${OS_MAJOR_VERSION}" == "9" ]; then echo -e "\033[1;36m=== Installing Oracle prerequisite packages for RHEL9 ===\033[0m"; REQUIRED_PACKAGES=(bc binutils compat-openssl11 elfutils-libelf fontconfig glibc glibc-devel ksh libaio libaio-devel libasan liblsan libX11 libXau libXi libXrender libXtst libxcrypt-compat libgcc libibverbs libnsl librdmacm libstdc++ libstdc++-devel libxcb libvirt-libs make gcc gcc-c++ policycoreutils policycoreutils-python-utils smartmontools sysstat); elif [ "${OS_MAJOR_VERSION}" == "8" ]; then echo -e "\033[1;36m=== Installing Oracle prerequisite packages for RHEL8 ===\033[0m"; REQUIRED_PACKAGES=(bc binutils elfutils-libelf elfutils-libelf-devel glibc glibc-devel ksh libaio libaio-devel libXrender libX11 libXau libXi libXtst libgcc libnsl librdmacm libstdc++ libstdc++-devel libxcb libibverbs make gcc gcc-c++ policycoreutils policycoreutils-python-utils smartmontools sysstat); else echo -e "\033[1;31m=== ERROR: Unsupported OS major version:  ${OS_MAJOR_VERSION} ===\033[0m"; exit 1; fi; for pkg in "${REQUIRED_PACKAGES[@]}"; do echo -e "\033[1;34m=== Installing ${pkg} ===\033[0m"; if ! dnf install -y -q "${pkg}" &>/dev/null; then echo -e "\033[1;31m=== ERROR:  Failed to install ${pkg} ===\033[0m"; exit 1; fi; done; echo -e "\033[1;32m=== All prerequisite packages installed successfully ===\033[0m"; echo -e "\033[1;36m=== Validating package installations ===\033[0m"; VALIDATION_FAILED=0; CRITICAL_PACKAGES=(gcc gcc-c++ make glibc-devel libstdc++-devel libaio-devel); echo -e "\033[1;35m=== Checking CRITICAL compilation packages ===\033[0m"; for pkg in "${CRITICAL_PACKAGES[@]}"; do if rpm -q "$pkg" &>/dev/null; then echo -e "=== \033[32mPASS\033[0m:   $pkg is installed ==="; else echo -e "=== \033[31mFAIL\033[0m:  $pkg is NOT installed (CRITICAL) ==="; VALIDATION_FAILED=1; fi; done; if gcc --version &>/dev/null; then GCC_VERSION=$(gcc --version | head -n1); echo -e "=== \033[32mPASS\033[0m: GCC is functional - ${GCC_VERSION} ==="; else echo -e "=== \033[31mFAIL\033[0m:   GCC is not functional ==="; VALIDATION_FAILED=1; fi; if make --version &>/dev/null; then MAKE_VERSION=$(make --version | head -n1); echo -e "=== \033[32mPASS\033[0m: make is functional - ${MAKE_VERSION} ==="; else echo -e "=== \033[31mFAIL\033[0m:  make is not functional ==="; VALIDATION_FAILED=1; fi; for pkg in "${REQUIRED_PACKAGES[@]}"; do if [[ " ${CRITICAL_PACKAGES[@]} " =~ " ${pkg} " ]]; then continue; fi; if [[ "$pkg" == "libibverbs" || "$pkg" == "librdmacm" ]]; then continue; fi; if dnf list installed "$pkg" &>/dev/null; then echo -e "=== \033[32mPASS\033[0m:  $pkg is installed ==="; else echo -e "=== \033[31mFAIL\033[0m: $pkg is not installed ==="; VALIDATION_FAILED=1; fi; done; if dnf list installed libibverbs &>/dev/null || dnf list installed mlnx-ofa_kernel &>/dev/null || rpm -q mlnx-ofa_kernel &>/dev/null; then echo -e "=== \033[32mPASS\033[0m:  libibverbs is installed (via mlnx-ofa_kernel or standard package) ==="; else echo -e "=== \033[31mFAIL\033[0m: libibverbs is not installed ==="; VALIDATION_FAILED=1; fi; if dnf list installed librdmacm &>/dev/null || dnf list installed mlnx-ofa_kernel &>/dev/null || rpm -q mlnx-ofa_kernel &>/dev/null; then echo -e "=== \033[32mPASS\033[0m: librdmacm is installed (via mlnx-ofa_kernel or standard package) ==="; else echo -e "=== \033[31mFAIL\033[0m: librdmacm is not installed ==="; VALIDATION_FAILED=1; fi; if [ ${VALIDATION_FAILED} -eq 1 ]; then echo -e "\033[1;31m=== ERROR:   Package validation failed ===\033[0m"; exit 1; fi; echo -e "\033[1;32m=== All packages validated successfully ===\033[0m"; echo -e "\033[1;36m=== Validating kernel and network parameters ===\033[0m"; PARAM_VALIDATION_FAILED=0; for param in "/proc/sys/kernel/shmmni: 4096" "/proc/sys/kernel/panic_on_oops:1" "/proc/sys/fs/file-max:6815744" "/proc/sys/fs/aio-max-nr:1048576" "/proc/sys/net/ipv4/ip_local_port_range:9000 65500" "/proc/sys/net/core/rmem_default:262144" "/proc/sys/net/core/rmem_max: 4194304" "/proc/sys/net/core/wmem_default:262144" "/proc/sys/net/core/wmem_max:1048576"; do param_path=$(echo "$param" | cut -d: -f1); expected_value=$(echo "$param" | cut -d: -f2- | xargs); actual_value=$(cat "$param_path" | xargs); if [[ "$param_path" == "/proc/sys/fs/file-max" && $actual_value -ge $expected_value ]] || [[ "$param_path" == "/proc/sys/net/ipv4/ip_local_port_range" && $(echo "$actual_value" | awk '"'"'{print $1}'"'"') == "9000" && $(echo "$actual_value" | awk '"'"'{print $2}'"'"') == "65500" ]] || [[ "$actual_value" == "$expected_value" ]]; then echo -e "=== \033[32mPASS\033[0m:  $param_path is $actual_value (expected: $expected_value) ==="; else echo -e "=== \033[31mFAIL\033[0m:   $param_path is $actual_value (expected: $expected_value) ==="; PARAM_VALIDATION_FAILED=1; fi; done; if [ ${PARAM_VALIDATION_FAILED} -eq 1 ]; then echo -e "\033[1;31m=== ERROR:  Kernel parameter validation failed ===\033[0m"; exit 1; fi; echo -e "\033[1;32m=== All kernel parameters validated successfully ===\033[0m"; echo -e "\033[1;36m=== Validating PAM configuration ===\033[0m"; PAM_FILE="/etc/pam.d/system-auth"; if grep -q "pam_limits.so" "${PAM_FILE}"; then echo -e "=== \033[32mPASS\033[0m: ${PAM_FILE} has pam_limits.so configured ==="; else echo -e "=== \033[31mFAIL\033[0m: ${PAM_FILE} missing pam_limits.so ==="; exit 1; fi; echo -e "\033[1;36m=== Validating Oracle user limits ===\033[0m"; if ! id oracle &>/dev/null; then echo -e "\033[1;31m=== FAIL:  Oracle user does not exist ===\033[0m"; exit 1; fi; ULIMIT_VALIDATION_FAILED=0; su - oracle -c "[[ \$(ulimit -Sn) -eq 1024 ]] && echo -e \"=== \033[32mPASS\033[0m:   Soft nofile: 1024 expected: 1024 ===\" || { echo -e \"=== \033[31mFAIL\033[0m: Soft nofile: \$(ulimit -Sn) expected: 1024 ===\"; exit 1; }; [[ \$(ulimit -Hn) -eq 65536 ]] && echo -e \"=== \033[32mPASS\033[0m:  Hard nofile: 65536 expected: 65536 ===\" || { echo -e \"=== \033[31mFAIL\033[0m:  Hard nofile: \$(ulimit -Hn) expected: 65536 ===\"; exit 1; }; [[ \$(ulimit -Su) -eq 16384 ]] && echo -e \"=== \033[32mPASS\033[0m: Soft nproc: 16384 expected: 16384 ===\" || { echo -e \"=== \033[31mFAIL\033[0m:   Soft nproc: \$(ulimit -Su) expected: 16384 ===\"; exit 1; }; [[ \$(ulimit -Hu) -eq 16384 ]] && echo -e \"=== \033[32mPASS\033[0m: Hard nproc: 16384 expected: 16384 ===\" || { echo -e \"=== \033[31mFAIL\033[0m: Hard nproc: \$(ulimit -Hu) expected: 16384 ===\"; exit 1; }; [[ \$(ulimit -Ss) -eq 10240 ]] && echo -e \"=== \033[32mPASS\033[0m:   Soft stack: 10240 expected: 10240 ===\" || { echo -e \"=== \033[31mFAIL\033[0m:   Soft stack: \$(ulimit -Ss) expected: 10240 ===\"; exit 1; }; [[ \$(ulimit -Hs) -eq 32768 ]] && echo -e \"=== \033[32mPASS\033[0m: Hard stack: 32768 expected: 32768 ===\" || { echo -e \"=== \033[31mFAIL\033[0m:   Hard stack: \$(ulimit -Hs) expected: 32768 ===\"; exit 1; }" || ULIMIT_VALIDATION_FAILED=1; if [ ${ULIMIT_VALIDATION_FAILED} -eq 1 ]; then echo -e "\033[1;31m=== ERROR:  Oracle user ulimit validation failed ===\033[0m"; exit 1; fi; echo -e "\033[1;32m=== All Oracle user limits validated successfully ===\033[0m"; echo -e "\033[1;32m=== Oracle 19c Prerequisites Installation and Validation Completed Successfully ===\033[0m"'
```

## Filesystem and Directory Setup

### Create Required Directories

Under root user, create `$ORACLE_HOME`, data, and recovery directories.

**Note:** In a real production environment, all of these should be actual filesystems. Follow this guide to order the disks/filesystems: https://confluence.shared.int.tds.tieto.com/display/DBPS/Oracle+OS+configuration

Switch to root user:
```bash
sudo su -
```

Create directories:
```bash
mkdir -p /oracle/app/oracle/product/19/dbhome_1 /oracle/oradata /oracle/oraarch /oracle/redologA /oracle/redologB /oracle/backup
```

Set ownership:
```bash
chown -R oracle:oinstall /oracle
```

### Prepare Installation Package

Give the package full permissions:
```bash
chmod 777 package.zip
```

Switch to oracle user:
```bash
sudo su - oracle
```

Define Oracle home location:
```bash
export ORACLE_HOME=/oracle/app/oracle/product/19/dbhome_1
```

Unzip the package under oracle user so all files are oracle-owned:
```bash
unzip <zip_path> -d $ORACLE_HOME
```

### Configure Database Edition

Define the edition of the database:
- For Enterprise Edition (EE): `EE`
- For Standard Edition 2 (SE2): `SE2`

```bash
export ORACLE_EDITION=EE
```

### Create Response File

Backup the original response file and create a new one:
```bash
mv $ORACLE_HOME/install/response/db_install.rsp $ORACLE_HOME/install/response/db_install.rsp_backup ; { echo "oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v19.0.0"; echo "oracle.install.option=INSTALL_DB_SWONLY"; echo "INVENTORY_LOCATION="${ORACLE_HOME%/*/*/*/*}"/oraInventory"; echo "ORACLE_HOME="$ORACLE_HOME""; echo "ORACLE_BASE="${ORACLE_HOME%/*/*/*}""; echo "oracle.install.db.InstallEdition=${ORACLE_EDITION}"; echo "oracle.install.db.OSDBA_GROUP=dba"; echo "oracle.install.db.OSOPER_GROUP=dba"; echo "oracle.install.db.OSBACKUPDBA_GROUP=dba"; echo "oracle.install.db.OSDGDBA_GROUP=dba"; echo "oracle.install.db.OSKMDBA_GROUP=dba"; echo "oracle.install.db.OSRACDBA_GROUP=dba"; echo "oracle.install.db.rootconfig.executeRootScript=false"; } > $ORACLE_HOME/install/response/db_install.rsp ; cat $ORACLE_HOME/install/response/db_install.rsp
```

## RHEL 9 Specific Requirements

**Important:** RHEL 9 requires Oracle 19.19 or later Release Update (RU) to be applied.

### Documentation References

- [Oracle Support Document 2982833.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=127815299033366&id=2982833.1&_afrWindowMode=0&_adf.ctrl-state=6u7agjics_4)
- [Supported Red Hat Enterprise Linux 9 Distributions](https://docs.oracle.com/en/database/oracle/oracle-database/19/lacli/supported-red-hat-enterprise-linux-9-distributions-for-x86-64.html)
- [Oracle Support Knowledge Article](https://support.oracle.com/knowledge/Oracle%20Cloud/2982833_1.html)

## Apply Oracle Patch (Required for RHEL 9)

### Prepare Environment

Login as oracle user and set `ORACLE_HOME`:
```bash
sudo su - oracle
```

```bash
export ORACLE_HOME=/oracle/app/oracle/product/19/dbhome_1
```

### Upgrade OPatch

Backup the existing OPatch and install the new version:
```bash
mv $ORACLE_HOME/OPatch $ORACLE_HOME/OPatch_backup_$(date +'%Y_%m_%d_%H_%M') ; ls -l $ORACLE_HOME/ | grep OPatch
```

Unzip the new OPatch:
```bash
unzip <location_of_zip> -d $ORACLE_HOME
```

Verify OPatch version:
```bash
$ORACLE_HOME/OPatch/opatch version
```

### Apply Database Patch

Unzip the database patch:
```bash
unzip <DB_patch_zip>
```

Set distribution ID for compatibility:
```bash
export CV_ASSUME_DISTID=RHEL8
```

Run the installer with the patch:
```bash
nohup $ORACLE_HOME/runInstaller -applyRU /oracle/database_release/patch/36582781 -silent -ignorePrereqFailure -waitforcompletion -responseFile /oracle/app/oracle/product/19/dbhome_1/install/response/db_install.rsp > runInstaller_swonly_$(date +'%Y_%m_%d_%H_%M').log &
```