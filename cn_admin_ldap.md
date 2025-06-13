# Como colocar o `CN = ADMIN` na Arvore do Ldap

### 1.0. Crie um arquivo .ldif com as configurações do user admin:

```
sudo vi cnadmin.ldif
```

- Adicione o conteudo abaixo
   
```
dn: cn=admin,dc=meusite,dc=com
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword: secret
```
### 2.0. Roda o comando para adicionar o user ao database do Ldap:

```
$ sudo ldapadd -x -D cn=admin,dc=meusite,dc=com -W -f base.ldif
```
