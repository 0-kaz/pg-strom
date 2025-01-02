PG-Strom
========
PG-Strom is an extension for PostgreSQL database.
It is designed to accelerate mostly batch and analytics workloads with
utilization of GPU and NVME-SSD, and Apache Arrow columnar.

For more details, see the documentation below.
http://heterodb.github.io/pg-strom/

Software is fully open source, distributed under PostgreSQL License.

---

PG-Strom + GPU Direct Storage Quickstart
========================================
���̃h�L�������g��PG-Strom�̓�������AGPU Direct Storage�܂ł̐ݒ�����������̂ł��B
�{����Local NVMe SSD��GPU Direct Storage�����ɗ��p������������Ă��܂��B

RHEL9�����ɏ�����Ă��܂����A[RHEL]�ƌ��o���ɏ�����Ă��镔���ȊO��Rocky Linux�Ȃǂ�RHEL�N���[��OS�ƊT�ˋ��ʂȂ̂ŎQ�l�ɂȂ�Ǝv���܂��B

����ł͂��̃K�C�h�ɏ]���Ċ��\�z���āAPG-Strom�̐��E�ɑ��𓥂ݓ���Ă݂Ă��������B


## [RHEL]�o�[�W�����Œ�

RHEL�ł̓����[�X�o�[�W������ύX�ł��܂��B�C���X�g�[������CUDA�ɍ��킹�āA�����[�X�o�[�W�������Œ艻����Ɨǂ��ł��傤�BCUDA��MOFED�ALinux�̃o�[�W������K�؂����R�ɑI���ł���ꍇ�͂��̐ݒ�͕s�v�ł��B

(rhel9)

```
$ sudo subscription-manager release --set=9.4
```

## �\�t�g�E�F�A�A�b�v�f�[�g�̎��{
�\�t�g�E�F�A�A�b�v�f�[�g�����s���܂��B

```
$ sudo dnf update -y
```

## [RHEL]EUS���|�W�g���[�̗L����
RHEL�ł̓V�X�e���ւ̉����X�V�T�|�[�g (EUS)�����p�ł��܂��BEUS�͒ʏ�̃T�|�[�g���Ԃ�蒷�����ԁA�}�C�i�[�o�[�W�����̃����e�i���X�A�b�v�f�[�g���󂯂��܂��B�����Ԃ̈��肵�����p��K�v�Ƃ���ꍇ�͓K�p���Ă��������BCUDA��MOFED�ALinux�̃o�[�W������K�؂����R�ɑI���ł���ꍇ�͂��̐ݒ�͕s�v�ł��B

(rhel9)

```
$ sudo subscription-manager repos --enable rhel-9-for-x86_64-appstream-eus-rpms --enable rhel-9-for-x86_64-baseos-eus-rpms
```

EPEL��CodeReady Linux Builder�iPowerTools�j�̗L����
EPEL�̗��p�ɂ͊J���Ŏg���p�b�P�[�W�Z�b�g���|�W�g���[�iCodeReady Linux Builder�j�̗L�������K�v�ł��B�f�B�X�g���r���[�V�����ɂ���āA���|�W�g���[�̖��̂��قȂ邱�Ƃ�����܂��B

�ڍׂ̓A�b�v�X�g���[���̃h�L�������g���m�F���Ă��������B

- https://docs.fedoraproject.org/en-US/epel/getting-started/

(rhel9)

```
$ sudo subscription-manager repos --enable codeready-builder-for-rhel-9-$(arch)-rpms && sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

## �J���c�[���Ȃǂ̃C���X�g�[��

```
$ sudo dnf install wget git-core -y
$ sudo dnf groupinstall 'Development Tools' -y
```

## IOMMU�̖�����

(rhel9)

```
$ sudo vi /etc/default/grub
  GRUB_CMDLINE_LINUX="crashkernel=auto iommu=off intel_iommu=off"
  GRUB_CMDLINE_LINUX_DEFAULT="rd.auto=1 iommu=off intel_iommu=off"

$ sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

## Nouveau�h���C�o�[�̖�����

```
# cat > /etc/modprobe.d/disable-nouveau.conf <<EOF
blacklist nouveau
options nouveau modeset=0
EOF
```

## �V�X�e���ċN��

```
# shutdown -r now
```

## �J�[�l���w�b�_�[�Ȃǂ̃C���X�g�[��
�ċN����ɁA���ݗ��p���Ă���Linux�J�[�l���o�[�W�����p�̃w�b�_�[�Ȃǂ��C���X�g�[�����܂��B

```
$ sudo dnf install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

## MOFED�̃C���X�g�[��
�_�E�����[�hURL����A���p�\���CUDA�o�[�W������Linux�f�B�X�g���r���[�V�����ALinux�J�[�l���̃o�[�W�����ɑΉ�����MOFED���_�E�����[�h���܂��B

- https://network.nvidia.com/products/infiniband-drivers/linux/mlnx_ofed/

���O�ɕK�v�ȃp�b�P�[�W���C���X�g�[������B����RHEL9�̗�B�o�[�W�����ɂ���ĕK�v�ȃp�b�P�[�W�͈قȂ�܂��B�r���h���ɕs�����Ă���Ƃ������b�Z�[�W���o����A�p�b�P�[�W��ǉ����܂��B

(rhel9)

```
$ sudo dnf install kernel-modules-extra kernel-rpm-macros lsof pciutils createrepo tk tcl gcc-gfortran perl-sigtrap
```

�W�J���ăr���h&�C���X�g�[�����܂��B

(rhel9)

```
$ sudo mount MLNX_OFED_LINUX-23.10-3.2.2.0-rhel9.4-x86_64.iso /mnt
$ cd  /mnt
$ sudo ./mlnxofedinstall --with-nvmf --with-nfsrdma --enable-gds --add-kernel-support && sudo dracut -f
```

## DNF�ݒ�ύX
MOFED�p�b�P�[�W���V�X�e���̃p�b�P�[�W�ŏ㏑�����Ȃ��悤�ɁAexcludepkgs�Ŏw�肵�܂��B

```
$ sudo vi /etc/dnf/dnf.conf

[main]
...
excludepkgs=mstflint,openmpi,perftest,mpitests_openmpi,mlnx-*,ofed-*
```

`ofed_rpm_info`�R�}���h�ŏo�Ă���p�b�P�[�W��u�������Ȃ��悤�ɐݒ肷�邱�ƁI
MOFED�̃o�[�W�����A�b�v��폜���ɂ͎�菜�����ƁB

��U�A�ċN�����܂��B

```
$ sudo reboot
```


## Local NVMe����̐ݒ�
MOFED�ŃC���X�g�[���������W���[�����m�F���܂��B`extra`�ȉ��̃p�X�ɂ���΁ALinux�J�[�l���O���̃��W���[�����g�����Ԃł��B

```
$ modinfo nvme
filename:       /lib/modules/5.14.0-427.31.1.el9_4.x86_64/extra/mlnx-nvme/host/nvme.ko
version:        1.0
license:        GPL
author:         Matthew Wilcox <willy@linux.intel.com>
rhelversion:    9.4
...
```

���W���[����ǂݍ��݂܂��B

```
$ sudo su -
# modprobe nvme
# echo nvme > /etc/modules-load.d/nvme.conf
```

�f�o�C�X�̐����m�F���܂��B

```
$ ls /dev |grep nvme
```

�f�o�C�X�̐��ɍ��킹�āA�\�t�g�E�F�ARAID�̐ݒ�(-n��NVMe SSD�f�o�C�X�����w��)���s���܂��B
 
```
# mdadm -C /dev/md0 -c 128 -l 0 -n 4 /dev/nvme0n1 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
# mdadm --detail --scan > /etc/mdadm.conf
```

RAID �{�����[���� udev ���[����`--no-devices`���w�肳�ꂽ�s�̓��e���C�����܂��B

```
# vi /lib/udev/rules.d/63-md-raid-arrays.rules
...
#IMPORT{program}="/usr/sbin/mdadm --detail --no-devices --export $devnode"
IMPORT{program}="/usr/sbin/mdadm --detail --export $devnode"
```

parted�R�}���h���g���āA�p�[�e�B�V�����쐬���s���܂��B�t�@�C���V�X�e����ext4���w�肵�܂��B

```
# parted /dev/md0
(parted) p                                                                
Error: /dev/md0: unrecognised disk label
Model: Linux Software RAID Array (md)                                     
Disk /dev/md0: 4001GB
...
(parted) mklabel gpt                                                      
(parted) mkpart                                                           
Partition name?  []? "PG-Strom Storages"                                  
File system type?  [ext2]? ext4                                           
Start? 0%
End? 100%                                                                 
(parted) p                                                                                                             
Model: Linux Software RAID Array (md)
Disk /dev/md0: 4001GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name               Flags
 1      1049kB  4001GB  4001GB  ext4         PG-Strom Storages

(parted) quit                                                             
Information: You may need to update /etc/fstab.
```

�t�@�C���V�X�e�����쐬���܂��B

```
# mkfs.ext4 /dev/md0p1
```

�}�E���g���܂��B�X�g���[�W���i�������邽�߂ɁA`/etc/fstab`�ɋL�q���܂��B

```
# mkdir -p /opt/nvme
# mount /dev/md0p1 /opt/nvme   
# vi /etc/fstab
...
/dev/md0p1  /opt/nvme ext4 data=ordered 0 0
```

## CUDA�̃C���X�g�[��
PG-Strom 5.x��CUDA 12.2�ȍ~�������_�̍Œ�v���ɂȂ�܂��B�K�؂�OS��J�[�l���o�[�W������p�ӂ�����őΉ�����CUDA���C���X�g�[�����܂��B

- https://developer.nvidia.com/cuda-toolkit-archive


���O��dkms�p�b�P�[�W�����Ă����ƕ֗��ł��B���̃p�b�P�[�W��EPEL���|�W�g���[�ɂ���܂��B

```
$ sudo dnf install -y dkms
```

���|�W�g���[��ǉ����܂��B

(rhel9)

```
$ sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel9/x86_64/cuda-rhel9.repo
```

�p�b�P�[�W��CUDA���C���X�g�[������Ƃ��́A�o�[�W�������w�肵�܂��B

```
sudo dnf install -y cuda-toolkit-12-3
```

GPU�h���C�o�[���C���X�g�[�����܂��BCUDA�o�[�W�����ɂ���ă��K�V�[�h���C�o�[��Open�h���C�o�[�����݂��܂����ACUDA 12.3�ȍ~��Open�h���C�o�[���C���X�g�[������Ă��Ȃ���GDS�����p�ł��܂���B�܂��AOpen�h���C�o�[��Turing����ȍ~��GPU�ɂ����Ή����Ă��܂���̂ŁA�Ή�����GPU��p�ӂ��邩�ACUDA 12.2GA��������12.2 Update1�܂ł̃o�[�W���������p�\��GPU�Ɗ���p�ӂ��Ă��������B

```
sudo dnf module install -y nvidia-driver:open-dkms
```

Note:
CUDA 12.2 Update2�ȍ~��GDS�̎d�l�ύX��GPU��GPU�h���C�o�[�̑g�ݍ��킹���d�v�ɂȂ�BTuring�����iP,V�j�̐���ł�12.2 Update1�𗘗p���邱�ƁBTuring�ȍ~�ł�Open GPU�h���C�o�[���K�v�B

MOFED����ꂽ���ƁACUDA�Ɠ����o�[�W������nvidia-gds�p�b�P�[�W���C���X�g�[�����܂��B

```
sudo dnf install -y nvidia-gds-12-3
```

## PostgreSQL�̃C���X�g�[��
RHEL�����RHEL�N���[��OS��PostgreSQL�p�b�P�[�W��񋟂��Ă��܂����A�񋟂����p�b�P�[�W��r���h�|���V�[���قȂ邽�߂ɁA�኱����ɈႢ���������邱�Ƃ�����܂��B�{���ł�PostgreSQL�R�~���j�e�B���񋟂���p�b�P�[�W�𗘗p����O��ŉ�����Ă��܂��B

- https://www.postgresql.org/download/

(rhel9)

```
$ sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

PostgreSQL�p�b�P�[�W�̃C���X�g�[�������܂��B�{�������������_�ł�PostgreSQL 15�ʈȍ~�̃o�[�W�����ɑΉ����Ă��܂��BPostgreSQL 16���C���X�g�[������ꍇ�͎��̂悤�Ɏ��s���܂��B

```
$ sudo dnf -qy module disable postgresql && sudo dnf install -y postgresql16-server postgresql16-devel
```

PostgreSQL�̏������ƃT�[�r�X�̋N����ݒ肵�܂��B

```
$ sudo /usr/pgsql-16/bin/postgresql-16-setup initdb && sudo systemctl enable --now postgresql-16
```

SELinux��������s���܂��B���ۗ��p����p�X�ɒu�������Ď��s���Ă��������B

```
$ sudo chown postgres:postgres -R /opt/nvme && sudo chcon -R system_u:object_r:postgresql_db_t:s0 /opt/nvme/
```

## HereroDB���|�W�g���[�̒ǉ�
HeteroDB Software Distribution Center���烊�|�W�g���[RPM�p�b�P�[�W���_�E�����[�h���āA`heterodb-extra`�p�b�P�[�W���C���X�g�[�����܂��B

- [HeteroDB Software Distribution Center](https://heterodb.github.io/swdc/)

(rhel9)

```
$ sudo dnf install -y https://heterodb.github.io/swdc/yum/rhel9-noarch/heterodb-swdc-1.3-1.el9.noarch.rpm
$ sudo dnf install -y heterodb-extra
```

���肵�����C�Z���X�����蓖�Ă܂��B

```
$ sudo sh -c "cat heterodb.license > /etc/heterodb.license"
$ sudo systemctl restart postgresql-16
```


## PG-Strom �̃C���X�g�[��
�����܂ŏ������ł�����A[�C���X�g�[���K�C�h](https://heterodb.github.io/pg-strom/ja/install/)�ɏ]���āAPG-Strom�̃C���X�g�[���A�ݒ���s���܂��B


## ���̑��̐ݒ�
�M�҂��悭�s���ݒ���܂Ƃ߂܂����B�K�v�ɉ����Đݒ肵�܂��B

### PostgreSQL + PG-Strom�����O���N���C�A���g���痘�p����
�O���N���C�A���g���痘�p����PG-Strom���փA�N�Z�X�������ꍇ�́APostgreSQL�̐ݒ�̕ύX���K�v�ł��B
���̂悤�ȕ��@�őΉ����Ă��������B

���b�X���A�h���X�̐ݒ��ύX���܂��B

```
sudo su - postgres
vi /var/lib/pgsql/16/data/postgresql.conf
...
listen_addresses = '*'
```

�����[�g�A�N�Z�X�������郆�[�U�[���쐬���܂��B���[�U�[�ɂ̓A�N�Z�X�ɓK�؂ȃ��[����ݒ肵�܂��B�ȉ��͂��Ȃ�ɂ��ݒ�ł��B�ݒ���e�̏ڍׂɂ��Ă͐ݒ�t�@�C���̃R�����g���m�F���Ă��������B

```
$ createuser -d -r -s -P pguser01
Enter password for new role:  <�p�X���[�h��ݒ�>
��vi /var/lib/pgsql/16/data/pg_hba.conf
�i�ǋL�j
host    all    pguser01    127.0.0.1/32     scram-sha-256
host    all    pguser01    172.16.0.0/16    scram-sha-256
host    all    pguser01    172.17.0.0/16    scram-sha-256
$ exit
```

�T�[�r�X���ċN�����܂��B

```
$ sudo systemctl restart postgresql-16.service
$ journalctl -u postgresql-16
```

�t�@�C�A�E�H�[���|�[�g�J��
�����[�g�A�N�Z�X��������ɂ̓|�[�g�J�����K�v�ł��BFirewalld�����p����Ă�����ł͈ȉ��̂����ꂩ�̕��@�ŕK�v�ȃ|�[�g��������܂��B�ݒ肵�Ă���|�[�g��5432�ł͂Ȃ��ꍇ�́A�K�؂ȃ|�[�g���w�肵�Ă��������B


```
$ sudo firewall-cmd --add-service=postgresql --permanent 

or

$ sudo firewall-cmd --add-port=5432/tcp --permanent 

then

$ sudo firewall-cmd --reload
```


### GDS�̈�Ƀf�[�^�x�[�X���쐬����
PG-Strom�ɂƂ��āAGPUDirect Storage�͏d�v�ȃR���|�[�l���g�̈�ł��B
GPUDirect Storage�̊��������Ă��APostgreSQL�̃f�[�^��GPUDirect Storage���L���ȃX�g���[�W�ɂȂ��ƁA�\���Ȑ��\���o�����Ƃ��ł��܂���B���̂悤�ȕ��@�Ńf�[�^�x�[�X�ƃe�[�u�����쐬������ŁA�f�[�^����舵���Ă��������B

```
CREATE TABLESPACE nvme LOCATION '/opt/nvme';
CREATE DATABASE testdb TABLESPACE nvme;
```

�e�[�u����Ԃɂ��Ă̏ڍׂ́A�A�b�v�X�g���[���̃h�L�������g���m�F���Ă��������B

- https://www.postgresql.jp/document/16/html/manage-ag-tablespaces.html

