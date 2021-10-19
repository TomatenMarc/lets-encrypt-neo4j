# Let's Encrypt Neo4j

Since it is always a good idea to encrypt your applications, but very few do, this project will deal with the encryption
of [Neo4j](https://neo4j.com/) with [Let's Encrypt](https://letsencrypt.org/de/) certificates which are generated
via [Certbot](https://certbot.eff.org/). Neo4j is set up so its internal browser is accessible via `HTTPS` and the
database by direct requests with `Bolt+S (wss)`.

Following this project, you will be able to host and securely access the community edition of Neo4J remotely. For this
purpose, [Docker](https://www.docker.com/) and [Docker-Compose](https://docs.docker.com/compose/) will be used to make
the project as close as possible to production mode on an external server.