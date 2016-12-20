# openldap
openldap+fusiondirectory
```bash
yum update && yum install epel-release
yum install -y openldap openldap-clients openldap-servers migrationtools

slappasswd -s passwd -n > /etc/openldap/passwd
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown ldap. /var/lib/ldap/*

systemctl enable --now slapd
mkdir ldap_config && cd ldap_config

cat <<'EOF'> ch_rootPW.ldif
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}
EOF

ldapadd -Y EXTERNAL -H ldapi:/// -f ch_rootPW.ldif

cat <<'EOF'> ch_domainSettings.ldif
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=sk2,dc=su" read by * none

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
add: olcRootPW
olcRootPW: passwd

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=Manager,dc=sk2,dc=su" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=sk2,dc=su" write   by * read
EOF

ldapmodify -Y EXTERNAL -H ldapi:/// -f ch_domainSettings.ldif
