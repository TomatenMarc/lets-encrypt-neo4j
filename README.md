# Let's Encrypt Neo4j

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
`neo4j.conf`. For this the occurrences of the `<your-domain>` must be changed according to the certified domain.

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
`.env.neo4j` we will create the authentication by specifying `<password>`. Thereby the username neo4j cannot be changed,
because it is expected by the system by default.

```
NEO4J_AUTH=neo4j/<password>
```