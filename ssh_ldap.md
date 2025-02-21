# configurar ssh com usuario ldap

**Adicione o endereço do servidor LDAP ao '/etc/hosts' arquivo se você não tiver um servidor DNS ativo na sua rede.**

```bash
sudo vim /etc/hosts
```
> 192.168.18.50 ldap.example.com

**Instale os utilitários do cliente LDAP no seu sistema Ubuntu:**

```
sudo apt install ldap-auth-client
```

## **Comece a configurar para que fiquem como as configurações a abaixo**

**1. Definir URI LDAP - Pode ser endereço IP ou nome do host**

![Captura de tela de 2025-02-21 12-04-23](https://github.com/user-attachments/assets/29647042-532c-4244-86ff-28d11a9f80cc)

**2. Defina um nome distinto para a base de pesquisa**

![Captura de tela de 2025-02-21 12-05-29](https://github.com/user-attachments/assets/1cf921a1-4745-4c28-aaf1-18211ca27b75)

**3. Selecione a versão  3 do LDAP**

![Captura de tela de 2025-02-21 12-07-05](https://github.com/user-attachments/assets/33d4d968-7c91-4ea2-9fa0-76195654a689)

**4. Selecione  Sim  para Make local root Database admin**

![Captura de tela de 2025-02-21 12-13-05](https://github.com/user-attachments/assets/0e4be4ca-042b-46c0-ba6b-f389a6db4fbf)

**5. Responda  Não  para Does the LDAP database require login?**

![Captura de tela de 2025-02-21 12-14-39](https://github.com/user-attachments/assets/c631e84d-3591-412b-8601-271808e11ce4)

**6. Defina uma conta LDAP para root, algo como cn=admin,cd=example,cn=com**

![Captura de tela de 2025-02-21 12-15-55](https://github.com/user-attachments/assets/babe4eef-bea0-4516-a332-ec17c7859c25)

**7. Forneça a senha da conta raiz LDAP**

![Captura de tela de 2025-02-21 12-17-27](https://github.com/user-attachments/assets/d12d31bd-f504-4245-b4ac-033380a4d757)

## Após a instalação, edite  /etc/nsswitch.confe 

**Adicione a autenticação ldap às  linhas passwd e  group .**

```
sudo vi /etc/nsswitch.conf
```

```bash
 # /etc/nsswitch.conf
 #
 # Example configuration of GNU Name Service Switch functionality.
 # If you have the `glibc-doc-reference' and `info' packages installed, try:
 # `info libc "Name Service Switch"' for information about this file.

 passwd:         compat systemd ldap
 group:          compat systemd ldap
 shadow:         compat ldap
 gshadow:        files
 
 hosts:          files mdns4_minimal [NOTFOUND=return] dns
 networks:       files
 
 protocols:      db files
 services:       db files
 ethers:         db files
 rpc:            db files
 
 netgroup:       nis
```
**Configure o arquivo ldap.conf**

```
sudo vi /etc/ldap.conf
```

```
host ldap.exemplo.com
base dc=exemplo,dc=net
ldap_version 3
scope sub
pam_filter objectclass=posixAccount
pam_password md5
nss_base_group ou=groups,dc=exemplo,dc=net

# se não tiver certificado no site remova essa opção abaixo
ssl on
tls_checkpeer no
```

**Configure o arquivo /etc/pam.d/common-password**

```
sudo vi /etc/pam.d/common-password
```

```
#
# /etc/pam.d/common-password - password-related modules common to all services
#
# This file is included from other service-specific PAM config files,
# and should contain a list of modules that define the services to be
# used to change user passwords.  The default is pam_unix.

# Explanation of pam_unix options:
# The "yescrypt" option enables
#hashed passwords using the yescrypt algorithm, introduced in Debian
#11.  Without this option, the default is Unix crypt.  Prior releases
#used the option "sha512"; if a shadow password hash will be shared
#between Debian 11 and older releases replace "yescrypt" with "sha512"
#for compatibility .  The "obscure" option replaces the old
#`OBSCURE_CHECKS_ENAB' option in login.defs.  See the pam_unix manpage
#for other options.

# As of pam 1.0.1-6, this file is managed by pam-auth-update by default.
# To take advantage of this, it is recommended that you configure any
# local modules either before or after the default block, and use
# pam-auth-update to manage selection of other modules.  See
# pam-auth-update(8) for details.

# here are the per-package modules (the "Primary" block)
password        requisite                       pam_pwquality.so retry=3
password        [success=3 default=ignore]      pam_unix.so obscure use_authtok try_first_pass yescrypt
password        sufficient                      pam_sss.so use_authtok
password        [success=1 user_unknown=ignore default=die]     pam_ldap.so try_first_pass
# here's the fallback if no module succeeds
password        requisite                       pam_deny.so
# prime the stack with a positive return value if there isn't one already;
# this avoids us returning an error just because nothing sets a success code
# since the modules above will each just jump around
password        required                        pam_permit.so
# and here are more per-package modules (the "Additional" block)
password        optional        pam_gnome_keyring.so
# end of pam-auth-update config
```

**Configure o arquivo /etc/pam.d/common-session**

```
sudo vi /etc/pam.d/common-session
```

```
#
# /etc/pam.d/common-session - session-related modules common to all services
#
# This file is included from other service-specific PAM config files,
# and should contain a list of modules that define tasks to be performed
# at the start and end of interactive sessions.
#
# As of pam 1.0.1-6, this file is managed by pam-auth-update by default.
# To take advantage of this, it is recommended that you configure any
# local modules either before or after the default block, and use
# pam-auth-update to manage selection of other modules.  See
# pam-auth-update(8) for details.

# here are the per-package modules (the "Primary" block)
session [default=1]                     pam_permit.so
# here's the fallback if no module succeeds
session requisite                       pam_deny.so
# prime the stack with a positive return value if there isn't one already;
# this avoids us returning an error just because nothing sets a success code
# since the modules above will each just jump around
session required                        pam_permit.so
# The pam_umask module will set the umask according to the system default in
# /etc/login.defs and user settings, solving the problem of different
# umask settings with different shells, display managers, remote sessions etc.
# See "man pam_umask".
session optional                        pam_umask.so
# and here are more per-package modules (the "Additional" block)
session required        pam_unix.so
session optional                        pam_sss.so
session optional                        pam_ldap.so
session optional        pam_systemd.so
session optional                        pam_mkhomedir.so
# end of pam-auth-update config
session optional        pam_mkhomedir.so skel=/etc/skel umask=077
```

**Configure o arquivo /etc/ldap/ldap.conf**

```
sudo vi /etc/ldap/ldap.conf
```

```
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.

BASE   dc=example,dc=com
URI    ldap://ldap.example.com 

#SIZELIMIT      12
#TIMELIMIT      15
#DEREF          never

# TLS certificates (needed for GnuTLS)
TLS_CACERT      /etc/ssl/certs/ca-certificates.crt
```
