# Authelia with Traefik
You need to customize the config for your domain and infrastructure

## docker-compose
```
./config/configuration.yml
```

```yml
version: '3'

services:
  authelia:
    image: authelia/authelia
    container_name: authelia
    volumes:
      - ./config:/config
    networks:
      - proxy
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.authelia.rule=Host(`authelia.example.com`)'
      - 'traefik.http.routers.authelia.entrypoints=websecure'
      - 'traefik.http.routers.authelia.tls=true'
      - "traefik.http.routers.authelia.tls.certresolver=letsencrypt"
      - 'traefik.http.middlewares.authelia.forwardauth.address=http://authelia:9091/api/verify?rd=https://authelia.example.com'
      - 'traefik.http.middlewares.authelia.forwardauth.trustForwardHeader=true'
      - 'traefik.http.middlewares.authelia.forwardauth.authResponseHeaders=Remote-User,Remote-Groups,Remote-Name,Remote-Email'
    expose:
      - 9091
    restart: unless-stopped
    healthcheck:
      disable: false

networks:
  proxy:
    external: true
                     
```





## config

### Authentication backend

#### LDAP
If you want to use LDAP as authentication backend.

```
./config/configuration.yml
```

```yml
authentication_backend:
  disable_reset_password: false
  refresh_interval: 5m
  ldap:
    implementation: custom
    url: ldap://openldap
    timeout: 5s
    start_tls: false
    tls:
      skip_verify: false
      minimum_version: TLS1.2
    base_dn: dc=ldap,dc=example,dc=com
    username_attribute: uid
    additional_users_dn: ou=users
    users_filter: (&({username_attribute}={input})(objectClass=person))
    additional_groups_dn: ou=groups
    groups_filter: (&(member={dn})(objectClass=groupOfNames))
    group_name_attribute: cn
    mail_attribute: mail
    display_name_attribute: displayName
    user: cn=admin,dc=ldap,dc=example,dc=com
    password: #secret 
```
Alternatively you can create a
```
./config/users_database.yml
```
in which you enter your users.

```yml
###############################################################
#                         Users Database                      #
###############################################################

# This file can be used if you do not have an LDAP set up.

# List of users
users:
  user1:
    displayname: "User1"
    # Password is Authelia
    password: "$argon2id$v=19$m=65536,t=1,p=8$bjRlVlg4T3QvdVNNT2hYSg$9gS10lSiFxdCdqZT0fKIdJofSsjpQmbLcIdyxpxwtJ0"
    email: user1@email.com
    groups:
      - admins
      - dev

#To hash password run: docker run authelia/authelia:latest authelia hash-password yourpassword
```

### Notifications

#### SMTP
If you want to send notification like password reset via email.
```
./config/configuration.yml
```

```yml
notifier:
  smtp:
    username: test@mydomain.com  #mail login 
  #   # This secret can also be set using the env variables AUTHELIA_NOTIFIER_SMTP_PASSWORD_FILE
    password: #password
    host: mail.privateemail.com #smtp server
    port: 465 #port
    sender: test@mydomain.com
    tls:
      minimum_version: TLS1.2
      skip_verify: false
```
 If you do ***not*** want to use ***smtp***
 Notifications are written to a txt file on the host
 ```yml
notifier:
  filesystem:
    filename: /config/notification.txt
 ```
