# whoami

A minimal(5.0MB+) docker image with a http server that echos hostname(identifies itself) with health check support.

This docker image can be used to demostrate and validate load balance ability of your service.

# Usage

## Run a simple server 

    $ docker run -d --name echo -p 80:8000 mpepping/whoami
    $ curl http://127.0.0.1
    2c47ad631c82

By default container exposes port `8000`, and returns container hostname(short id) as response.

## Control what is returned 

You can control what is returned by setting `MESSAGE` environment variable(the following example is illustrated under docker 1.12 swarm mode):

    $ docker service create -e MESSAGE="viola" -p 8000:8000 --replicas=2 whoami:v0.4
    $ curl http://127.0.0.1:8000
    viola from bf8cf715445d
    $ curl http://127.0.0.1:8000
    viola from 15d94216ff07

## Check the logs
And every container outputs requests to stdout, which is accessible by standard docker logs:

    $ docker logs bf8cf7
    2016/11/04 03:25:35 start serving...
    2016/11/04 03:25:51 [GET] at "/" takes 41.015µs
    2016/11/04 03:25:53 [GET] at "/" takes 26.656µs

## Health Check

`whoami` exposes a health check endpoint at `/health`, which tells you if the service is working well.
If the service is ok, it will return `ok` with `status 200`, otherwise it will return `Oops!` with `status 500`.
You can toggle the health status by send a post request to `/toggle.hz`:

    $ curl http://127.0.0.1:8007/health                  
    ok
    ➜  curl -X POST http://127.0.0.1:8007/toggle.hz 
    done.
    $ curl http://127.0.0.1:8007/health                
    Oops!%                          

**NOTE**: The health endpoint is only for test purpose, it does not affect other endpoints.

# Build

Create the binaries for the native os with `go build`:

    CGO_ENABLED=0 go build -a -ldflags '-s' .
    
Or cross-compiled for 64-bit Linux:

    CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -ldflags '-s' .


For more infromation about the parameters, please refer to [this blog](http://blog.xebia.com/create-the-smallest-possible-docker-container/).

Then, build the docker image:

    docker build -t whoami:v1.0 .

To copy an alternative Go binary at buildtime:

    docker build --build-arg TARGET=whoami-linux -t whoami:v1.0 .    
    

# Creds

Built on cizixs/whoami
