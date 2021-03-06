# Let's Encrypt Neo4j

[![License - MIT](https://img.shields.io/badge/License-MIT-2ea44f)](./LICENSE.md)
[![Author-GIT](https://img.shields.io/static/v1?label=GitHub&message=Tomatenmarc&color=blue&logo=github)](https://github.com/Tomatenmarc)
[![Author-TWITTER](https://img.shields.io/static/v1?label=Twitter&message=FegerMarc&color=blue&logo=twitter)](https://twitter.com/FegerMarc)

Since it is always a good idea to encrypt your applications, but very few do, this project will deal with the encryption
of [Neo4j](https://neo4j.com/) with [Let's Encrypt](https://letsencrypt.org/de/) certificates which are generated
via [Certbot](https://certbot.eff.org/). Neo4j is set up so its internal browser is accessible via `HTTPS` and the
database by direct requests with `Bolt+S (wss)`.

Following this project, you will be able to host and securely access the community edition of Neo4J remotely. For this
purpose, [Docker](https://www.docker.com/) and [Docker-Compose](https://docs.docker.com/compose/) will be used to make
the project as close as possible to production mode on an external server.

# Step 1: Receiving Certificates

In the first step, the certificates must be created, which can later be used for encryption by Neo4J. To do this,
Certbot must initially be created on the machine via Docker. If the latest version is already available, this step can
be skipped. To create Certbot, the following command should be used:

```
$ docker pull certbot/certbot:latest
```

Afterwards, real certificates can be created with Certbot. For this purpose, the following command must be used, whose
parameters are explained below.

```
$ docker run \
    -p "80:80" \
    -v "${PWD}/neo4j/certificates/etc/letsencrypt:/etc/letsencrypt" \
    -it \
    --name certbot \
    --rm \
      certbot/certbot certonly \
    --standalone \
    --email <your-email> \
    --agree-tos \
    -d <your-domain> \
    --renew-by-default
```

In this setup Certbot generates Let's Encrypt certificates without an own server
like [NGINX](https://github.com/wmnnd/nginx-certbot) on port `80` answering
the [HTTP-01 challenge](https://letsencrypt.org/de/docs/challenge-types/). This is done by using `--standalone` to
create a small server inside the container named `certbot`. Since the container runs on a server and the ports must be
opened by default from the container to the host, so that the container can be reached from outside, it is necessary to
release port `80` with `-p "80:80"`. Furthermore, the created certificates are stored
in `${PWD}/neo4j/certificates/etc/letsencrypt` via `-v "${PWD}/neo4j/certificates/etc/letsencrypt:/etc/letsencrypt"`. In
order for Certbot to send information about the certificates to the requester, an email address is requested
using `--email <your-email>`, in addition to the domain to certify `-d <your-domain>`. Overall, the conditions of
Certbot are agreed to with `--agree-tos` and already existing certificates are overwritten with `--renew-by-default`.

# Step 2: Preparing Neo4j

After the certificates have been successfully stored in `${PWD}/neo4j/certificates/etc/letsencrypt/live/<your-domain>/`
in preparation for Neo4j the certified domain under which the instance should be reachable has to be specified in
`neo4j.conf`. For this the occurrences of `<your-domain>` must be changed according to the certified domain.

Therefore, `neo4j.conf` must be modified along the following lines:

```
dbms.default_advertised_address=<your-domain>
```

```
dbms.ssl.policy.bolt.base_directory=/var/lib/neo4j/certificates/etc/letsencrypt/live/<your-domain>/
```

```
dbms.ssl.policy.https.base_directory=/var/lib/neo4j/certificates/etc/letsencrypt/live/<your-domain>/
```

Before Neo4j can finally be operated, the `.env.neo4j` file must be created in the top folder of this project. In
`.env.neo4j` we will create the authentication by specifying `<password>`. Thereby the username `neo4j` cannot be changed,
because it is expected by the system by default.

```
NEO4J_AUTH=neo4j/<password>
```

# Step 3: Starting Neo4j

In order to start the Neo4j instance, it is sufficient to have this project on a dedicated and configured server on
which the previous steps have been performed. To start the instance only the following command must be executed:

```
$ docker-compose up
```

Afterwards, the Neo4J instance can be reached via `https://<your-domain>:7473`. Likewise, a connection to the database
itself can be set up directly via `Bolt+S (wss)` at `<your-domain>:7687`. This then allows direct queries and
connections from other external clients to be established.

# Conclusion

With this project it is possible to host your own Neo4j instance on an external server in only three simple steps and thus
take the first step into the world of graph databases. All in all three important steps are needed to encrypt your Neo4j
instance:

1. Valid Let's Encrypt certificates for your own domain pointing to your server
2. A Neo4J configuration that uses these certificates for `HTTPS` and `Bolt+S (wss)` and enables authentication
3. A Docker-Compose file with which the previous steps are properly injected into the Neo4j instance