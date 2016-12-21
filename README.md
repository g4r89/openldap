# openldap
# centos7+openldap+fusiondirectory+nginx
```bash
yum install -y epel-release && yum -y update
yum -y install openldap-servers openldap-clients

cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
chown ldap. /var/lib/ldap/DB_CONFIG

sed -i.bak '/SELINUX/s/enforcing/disable/' /etc/selinux/config
systemctl disable --now firewalld
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

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=Manager,dc=sk2,dc=su" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=sk2,dc=su" write   by * read
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

yum install -y fusiondirectory fusiondirectory-schema schema2ldif

fusiondirectory-insert-schema -i /etc/openldap/schema/cosine.schema
fusiondirectory-insert-schema -i /etc/openldap/schema/inetorgperson.schema
fusiondirectory-insert-schema -i /etc/openldap/schema/nis.schema
fusiondirectory-insert-schema
```
# nginx + php-fpm + apc
```bash
yum install nginx php-fpm php-cli php-mysql php-gd php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-magickwand php-magpierss php-mbstring php-mcrypt php-mssql php-shout php-snmp php-soap php-tidy

yum install -y php-pecl-apc

cat <<'EOF'> /etc/nginx/conf.d/fd.conf
server {
  listen 80;
  root /usr/share/fusiondirectory/html;
  index index.php;
 
  server_name fusiondirectory.acme.com;
 
  location ~ ^/.*\.php(/|$) {
    expires off; # do not cache dynamic content
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
    fastcgi_param DOCUMENT_ROOT $realpath_root;
    include /etc/nginx/fastcgi_params; # see /etc/nginx/fastcgi_params
  }
}
EOF

vi /etc/php.ini
cgi.fix_pathinfo=0
expose_php=off

systemctl enable --now php-fpm
systemctl enable --now nginx
```
# install plugins
```bash
yum install -y fusiondirectory-plugin-ldapdump fusiondirectory-plugin-ldapmanager -y

yum install -y fusiondirectory-plugin-systems fusiondirectory-plugin-systems-schema fusiondirectory-plugin-argonaut-schema
fusiondirectory-insert-schema -i /etc/openldap/schema/fusiondirectory/service-fd.schema
fusiondirectory-insert-schema -i /etc/openldap/schema/fusiondirectory/systems-fd-conf.schema
fusiondirectory-insert-schema -i /etc/openldap/schema/fusiondirectory/systems-fd.schema
fusiondirectory-insert-schema -i /etc/openldap/schema/fusiondirectory/argonaut-fd.schema

yum install -y fusiondirectory-plugin-samba fusiondirectory-plugin-samba-schema
fusiondirectory-insert-schema -i /etc/openldap/schema/fusiondirectory/samba-fd-conf.schema
fusiondirectory-insert-schema -i /etc/openldap/schema/fusiondirectory/samba.schema

yum install -y fusiondirectory-plugin-mail fusiondirectory-plugin-mail-schema
fusiondirectory-insert-schema -i /etc/openldap/schema/fusiondirectory/mail-fd.schema
fusiondirectory-insert-schema -i /etc/openldap/schema/fusiondirectory/mail-fd-conf.schema
```

