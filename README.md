# openldap
centos7+openldap+fusiondirectory
```bash
yum -y update && yum install -y epel-release

gpg --keyserver keys.gnupg.net --recv-key 62B4981F 
gpg --export -a "Fusiondirectory Archive Manager <contact@fusiondirectory.org>" > FD-archive-key
cp FD-archive-key /etc/pki/rpm-gpg/RPM-GPG-KEY-FUSIONDIRECTORY
rpm --import  /etc/pki/rpm-gpg/RPM-GPG-KEY-FUSIONDIRECTORY

cat <<'EOF'> /etc/yum.repos.d/fusion.repo
[fusiondirectory]
name=Fusiondirectory Packages for RHEL / CentOS 7
baseurl=http://repos.fusiondirectory.org/rhel7/RPMS
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-FUSIONDIRECTORY

[fusiondirectory-extra]
name=Fusiondirectory Packages for RHEL / CentOS 7
baseurl=http://repos.fusiondirectory.org/rhel7-rpm-extra/RPMS/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-FUSIONDIRECTORY
EOF
