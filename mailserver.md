# Mail-Server-Configuration (Ubuntu 20.04)
Postfix+Dovecot

AWS

1 VPC (default is enough)
2 Instances (1 Server and 1 Client)
2 Elastic IPs ( 1 for Server 1 for Client)


Terminal (Termius/Putty)

Server

- First step change your hostname -> `sudo hostnamectl set-hostname mailsrv.computer.pt`

- Update the machine -> `apt update && apt -y upgrade` 

- Install postfix -> `apt -y install postfix postfix-doc dovecot-imapd dovecot-pop3d lisas12-devsas12-bin` -> Select 2nd option -> write your domain (in this case computer.pt)

- Reconfigurate postfix -> `dpkg-reconfigure postfix` - select 2nd option - put your domain - "ubuntu" - in this option dont change anything - "no" - in this option clean all - "0" - "+" - and select ipv4,ipv6 or both

`nano /etc/default/saslauthd`
```
START = yes
OPTIONS = “ copy line 54 and put it here”
```

## POSTFIX

`nano /etc/postfix/sasl/smtpd.conf`

```
put the next 2 lines inside

   pwcheck_method: saslauthd
   mech_list: PLAIN LOGIN
```

to enable STARTTLS and SSL go to - `nano /etc/postfix/master.cf` ❗
```
uncomment on the 1st group
     submission inet n       -       y       -       -       smtpd
     -o syslog_name=postfix/submission
     -o smtpd_tls_security_level=encrypt
     -o smtpd_sasl_auth_enable=yes
add this -> -o smtpd_client_restrictions=permit_sasl_authenticated,reject

   uncomment on 2nd group
   smtps     inet  n       -       y       -       -       smtpd
   -o syslog_name=postfix/smtps
   -o smtpd_tls_wrappermode=yes
   -o smtpd_sasl_auth_enable=yes
add this -> -o smtpd_client_restrictions=permit_sasl_authenticated,reject
```
- To add your certificates so you can use STARTTLS or SSL -> `nano /etc/postfix/main.cf` -> and change the names for the names of your certificates 
                                                                                                                                                   
```
smtpd_tls_cert_file=/etc/ssl/certs/______.crt
smtpd_tls_key_file=/etc/ssl/private/_____.key
```

## DOVECOT

- To give Authorization-> `nano /etc/dovecot/conf.d/10-auth.conf`
   ```
   disable_plaintext_auth = yes -> uncomment and change yes -> no
```

- To define for which Directory your mails will go ->`nano /etc/dovecot/conf.d/10-mail.conf`
```
   uncomment mail_location = maildir:~/Maildir
   comment -> mail_location= mbox:~/mail :INBOX =/var/mail/%u
```
- TO add your Certificates so you can use STARTTLS and SSL -> `nano /etc/dovecot/conf.d/10-ssl.conf` 
```
ssl_cert = </etc/ssl/certs/________.crt (change for the name of your certificate)
ssl_key = </etc/ssl/private/________.key (change for the name of your certificate's key)
```
- To create automatic the Maildir everytime you create a user 
                                                              
```                                                              
cd  /etc/skel
do the next command -> maildirmake.dovecot Maildir
```

- To give permitions to postfix -> `adduser postfix sasl` ❗
- To give permitions to dovecot -> `adduser dovecot sasl` ❗

- add 1 user `adduser azores`

- Restart the system -> `systemctl restart saslauthd postfix dovecot`

## Client

- First step change your hostname -> `sudo hostnamectl set-hostname mailcli.computer.pt` -> the same domain as the server

- Update the machine -> `apt update && apt -y upgrade` 

- Install a Graphic interface with thunderbird -> [Link](https://gist.github.com/jdmedeiros/32bbb759d74860d1de92c6a2c34f96f1) (go to this link) -> (ignore the adduser and the login)(replace filezilla with thunderbird)(on the first step use 2nd option)
- Add 1 user (the same as you created in the server)
```
adduser azores
cp /home/ubuntu/.xsession /home/azores/.xsession
chown azores:azores /home/azores/.xsession 
```

✔️ To check if its working ,login in the Graphic interface with the user you made -> open thunderbird create an email with the server domain after @ 
(ex azores@computer.pt) ->  and try to send a mail to your own email.




