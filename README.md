# openldap
centos7+openldap+fusiondirectory
```bash
yum install -y epel-release && yum -y update
yum -y install openldap-servers openldap-clients

cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
chown ldap. /var/lib/ldap/DB_CONFIG

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
olcAccess: {2}to * by dn="cn=Manager,dc=sk2,dc=su" write by * read
EOF

ldapadd -Y EXTERNAL -H ldapi:/// -f chdomain.ldif
