# Deploying using docker-compose

Creating base dirs

```
mkdir deploy
cd deploy
mkdir -p {mariadb,config}
chown 1001.1001 -R mariadb
```

Creating app.json

```
cat <<EOF >config/app.json
{
  "allow_create_new_accounts" : true,
  "send_emails"              : false,
  "application_sender_email" : "email@test.com",
  "email_transporter" : {
    "host" : "localhost",
    "port" : 25,
    "auth" : {
      "user" : "user",
      "pass" : "pass"
    }
  },
  "sessionStore": {
    "useRedis": false,
    "redisConnectionConfiguration": {
      "host": "localhost",
      "port": 6379
    }
  },
  "ga_analytics_on" : false,
  "crypto_secret" : "!2~HswpPPLa22+=±§sdq qwe,appp qwwokDF_",
  "application_domain" : "http://app.timeoff.management",
  "promotion_website_domain" : "http://timeoff.management",
  "locale_code_for_sorting": "en"
}
EOF
```

Creating db.json

```
cat <<EOF >config/db.json
{
  "development": {
    "dialect": "sqlite",
    "storage": "./db.development.sqlite",
    "logging": false
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": "superpasswordDB",
    "database": "timeoff",
    "host": "mariadb",
    "dialect": "mysql"
  }
}
EOF
```

Creating localisation.json

```
cat <<EOF >config/localisation.json
{}
EOF
```

Creating docker-compose.yml

```
cat <<EOF >docker-compose.yml
version: '3.5'

services:

  timeoff:
    image: blinkit-timeoff:v0
    restart: always
    ports:
      - "80:3000"
    volumes:
      - /ebs/deploy/config:/app/config
    environment:
      - NODE_ENV=production
    depends_on:
      mariadb:
        condition: service_healthy
    healthcheck:
      interval: 10s
      retries: 12
      test: curl --write-out 'HTTP %{http_code}' --fail --silent --output /dev/null http://localhost:3000/

  mariadb:
    image: bitnami/mariadb:10.8
    restart: always
    volumes:
      - /ebs/deploy/mariadb:/bitnami/mariadb
    environment:
      - MARIADB_ROOT_PASSWORD=superpasswordDB
      - MARIADB_DATABASE=timeoff
    healthcheck:
      interval: 5s
      retries: 12
      test: /opt/bitnami/scripts/mariadb/healthcheck.sh

EOF
```
