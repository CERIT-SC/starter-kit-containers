# Starter-kit-containers

## Funnel Docker image

The Containers implementation for GDI is based on the [TES API](https://github.com/ga4gh/task-execution-schemas) and [funnel](https://ohsu-comp-bio.github.io/funnel/) implementations, the funnel link contains also full funnel documentation. For GDI, it is required to use the forked funnel version from [CERIT-SC github repository](https://github.com/CERIT-SC/funnel-gdi), this version includess support for `htsget` protocol for data staging. The repository is needed to build the Docker image. Pre-built image is available at `cerit.io/gdi/funnel:v0.11`. 

To build from sources and build own Docker image, just type:
```
git clone https://github.com/CERIT-SC/funnel-gdi
cd funnel-gdi
docker build -t {registry/imagename} .
docker push -t {registry/imagename}
```

## Running the service

Best option for running the funnel container service is using [docker compose](https://docs.docker.com/compose/). Note that v2.17.3 or later of Docker Compose is required and it is tested version known to work.

To run the service, just type:
```
git clone https://github.com/CERIT-SC/starter-kit-containers.git
cd starter-kit-containers
docker-compose up 
```

Note that the service requires privileged container as it runs Docker in Docker which does not work without privileged mode.

## Testing service

The `examples` directory contains three tests, all are just simple bash scripts to run a task into the funnel. The first `example-hello.sh` is simple example without any input/ouput files to test standalone task submission. 

Invocation and expected response:
```
./examples/example-hello.sh
{"id":"ch9cdbm7ilkm719vmd4g"}
```

Task with id `ch9cdbm7ilkm719vmd4g` has been created.

The funnel container should print something like: 
```
starter-kit-containers-funnel-1  | {"attempt":0,"index":0,"level":"info","msg":"EXECUTOR_STDOUT","ns":"local","stdout":"TESK says: Hello World\n","taskID":"ch9cdbm7ilkm719vmd4g","time":"2023-05-03T20:40:17Z","timestamp":"2023-05-03T20:40:17.603416431Z"}
```

There are two more examples: `example-htsget.sh` and `example-auth.sh` both showing how to use `htsget` protocol and how to use authentication. However, for these two examples to work, it is needed to run [starter-kit-hstget](https://github.com/GenomicDataInfrastructure/starter-kit-htsget/tree/feature/minimal-htsget) and it is needed to be properly configured, mainly, replace the `localhost` and the `s3-archive` in [config](https://github.com/GenomicDataInfrastructure/starter-kit-htsget/blob/feature/minimal-htsget/config/config.json) with the hostname of the server where htsget shall run. Also, replace `{YOUR_HOSTNAME_HERE}` in the both examples with the same hostname. It can be run in the same way as the first example.
