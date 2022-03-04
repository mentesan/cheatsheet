# cheatsheet
Tips and tricks

LDAP

ldapsearch  -H ldap://172.7.6.5 -x -W -D "user@domain.local" -b "dc=domain,dc=local" "(sAMAccountName=user)"
LDAPTLS_REQCERT=never ldapsearch -Z -H ldap://172.5.6.7 -x -W -D "user@domain.local" -b "dc=domain,dc=local" "(sAMAccountName=user)"
