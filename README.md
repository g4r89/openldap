# openldap
openldap+fusiondirectory
```bash
yum update && yum install epel-release
yum -y install openldap-servers openldap-clients
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown ldap. /var/lib/ldap/DB_CONFIG

systemctl enable --now slapd
mkdir ldap_config && cd ldap_config
