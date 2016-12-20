# openldap
centos7+openldap+fusiondirectory
```bash
yum install -y epel-release && yum -y update
yum -y install openldap-servers openldap-clients

cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
chown ldap. /var/lib/ldap/DB_CONFIG

sed -i.bak '/SELINUX/s/enforcing/disable/' /etc/selinux/config
systemctl disable --now firewald
systemctl enable --now slapd

slappasswd

cat <<'EOF'> chrootpw.ldif
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}
EOF

ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif

cat <<'EOF'> chdomain.ldif
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=sk2,dc=su

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=sk2,dc=su

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by
  dn.base="cn=Manager,dc=sk2,dc=su" read by * none
EOF

ldapadd -Y EXTERNAL -H ldapi:/// -f chdomain.ldif

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

yum install -y fusiondirectory
yum install -y fusiondirectory-schema schema2ldif

fusiondirectory-insert-schema -i /etc/openldap/schema/cosine.schema
fusiondirectory-insert-schema -i /etc/openldap/schema/inetorgperson.schema
fusiondirectory-insert-schema -i /etc/openldap/schema/nis.schema
fusiondirectory-insert-schema
