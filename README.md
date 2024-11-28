# Opinionated and CI-Tested Boilerplate Configuration for MariaDB with TLS and Encryption

This boilerplate configuration is designed for MariaDB with a focus on security through TLS
and data encryption. The configurations have been tested on the following platforms:

- **Debian 11 (Bullseye)** - MariaDB 10.6
- **Debian 12 (Bookworm)** - MariaDB 10.11
- **Debian Testing** - MariaDB 11.5
- **Ubuntu 22.04 (Jammy Jellyfish)** - MariaDB 10.6
- **Ubuntu 24.04 (Noble Numbat)** - MariaDB 10.11

## TLS Configuration

MariaDB supports encrypting data in transit between the server and clients using the Transport
Layer Security (TLS) protocol. For more information, refer to the
[MariaDB documentation on securing connections](https://mariadb.com/kb/en/securing-connections-for-client-and-server/).

### Setting Up TLS with Easy-RSA

Easy-RSA is a command-line utility for building and managing a Public Key Infrastructure (PKI)
Certificate Authority (CA). Below is a step-by-step guide to set up TLS using Easy-RSA.
Note that the default options are used for demonstration purposes;
you should customize them as needed for production use.

```bash
# Create a directory for the PKI CA
make-cadir pki-ca
cd pki-ca

# Initialize the PKI
./easyrsa init-pki

# Build the CA without a password
./easyrsa --batch build-ca nopass

# Generate server and client certificates
./easyrsa --batch build-server-full mariadb-server nopass
./easyrsa --batch build-client-full mariadb-client1 nopass
./easyrsa --batch build-client-full mariadb-client2 nopass

# Create a directory for SSL certificates
mkdir -p /etc/ssl/mariadb

# Copy the generated certificates and keys to the SSL directory
cp pki/ca.crt /etc/ssl/mariadb/mariadb-ca.pem
cp pki/issued/mariadb-server.crt /etc/ssl/mariadb/mariadb-ssl.crt
cp pki/private/mariadb-server.key /etc/ssl/mariadb/mariadb-ssl.key
cp pki/issued/mariadb-client*.crt /etc/ssl/mariadb/
cp pki/private/mariadb-client*.key /etc/ssl/mariadb/

# Set appropriate permissions for the SSL directory
chmod a+r /etc/ssl/mariadb/*
```

## Data Encryption

Encrypting tables ensures that even if someone gains physical access to the hard disk,
they cannot retrieve the original data. For more details, see the
[MariaDB documentation on data-at-rest encryption](https://mariadb.com/kb/en/data-at-rest-encryption-overview/).

### Creating Encryption Keys

Generating encryption keys is straightforward. Follow the steps below:

```bash
# Create a directory for the encryption keys
mkdir -p /etc/ssl/mariadb
cd /etc/ssl/mariadb/

# Generate a plain key file and a password key file
(echo -n "1;"; openssl rand -hex 32) > mariadb-keyfile.plain
openssl rand -hex 128 > mariadb-pass.keyfile

# Encrypt the plain key file using AES-256-CBC
openssl enc -aes-256-cbc -md sha1 -pass file:mariadb-pass.keyfile -in mariadb-keyfile.plain -out mariadb-keyfile.encrypted
```
