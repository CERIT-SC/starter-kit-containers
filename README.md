# First time setup guide

## Create the global docker network

```bash
docker network create my-app-network
```

## Setup LSAAI Mock

Generate a self-signed certificate for the LSAAI nginx proxy using this command:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./lsaai-mock/configuration/nginx/certs/nginx.key -out ./lsaai-mock/configuration/nginx/certs/nginx.crt -addext subjectAltName=DNS:aai.localhost
```

## Setup REMS

1. Generate jwks

```bash
cd rems
python generate_jwks.py
cd ..
```

2. Initialize the database

```bash
./rems/migrate.sh
```

3. Run the `rems-app` service

```bash
docker compose up -d rems-app
```

4. Log in to the REMS application (`http://localhost:3000`), so your user gets created in the database
5. Initialize the application

```bash
./rems/setup.sh
```

At steps 2, 3 and 5 it is important not to `cd` into the `rems` directory.

## Run the rest of the services

```bash
docker compose up -d
```

## Uploading a dataset to SDA

In `upload.sh` there is a simple script that uploads a dataset to SDA.

### Prerequisites

- `sda-admin` and `sda-cli` in your `$PATH`
- access token from `http://localhost:8085/`
- C4GH public key generated (and printed on startup) in the `storage-and-interfaces/credentials` container
- `s3cmd.conf` file in the same directory as the script. You can just replace `<access_token>` in this example:

```bash
[default]
access_token = <access_token>
human_readable_sizes = True
host_bucket = http://localhost:8002
encrypt = False
guess_mime_type = True
multipart_chunk_size_mb = 50
use_https = False
access_key = jd123_lifescience-ri.eu
secret_key = jd123_lifescience-ri.eu
host_base = http://localhost:8002
check_ssl_certificate = False
encoding = UTF-8
check_ssl_hostname = False
socket_timeout = 30
```

### Usage

1. Create a file named `crypt4gh_key.pub` and copy the key from the `storage-and-interfaces/credentials` container output log
2. Move all your data to a folder next to the script
3. Run `./upload.sh <folder_name> <dataset_name>`
4. After successful upload, you should be able to fetch the data using:

```bash
token=<access_token>
curl -H "Authorization: Bearer $token" http://localhost:8443/s3/<dataset>/jd123_lifescience-ri.eu/<folder_name>/<file_name>
```

## Running a workflow from the compute-web

1. Log in to `http://localhost:4180`
2. Select a workflow on the home page
3. Define the input directory as `<dataset>/<user_id>` (e.g. `DATASET001/jd123_lifescience-ri.eu`)
4. Define the output directory. This is the directory in the s3 inbox bucket (e.g., `myWorklowOutput`)
5. Click `Run`
