Pravah.io
=========
![](https://github.com/pravahio/go-mesh/workflows/Go/badge.svg)

Pravah is an open source data exchange platform for smart cities with a focus on high velocity or real-time data.

We envision to connect different stakeholders in a smart city like data producers and consumers to each other to enable a data economy, improve collaboration and simplify access to real-time data. High data quality, low latency, well-defined data sharing standards and easy deployability across different platforms are some of the goals we are looking towards. This would enable anyone to build integrated mobility applications from multi-modal route planning to inter-connected traffic signals managing traffic using AI.

## Prerequisite

1. git
2. go == v1.12.*
3. make
4. gcc
_____________
## Installation

**Note:** You need not build it before installing. Program is build and installed in a single step.

```
git clone https://github.com/pravahio/go-mesh.git
cd go-mesh
make install
```
__________________

## Creating a new account

An account is a public-private key pair which will be used to sign all the messages and transactions flowing out of your node.

You can create one by using the following command

```
mesh account -c -p -o account.msa
```

____________

## Config file

`mesh` has many configuration options all of which are available in `mesh --help`.

Instead of passing them as flags we can use a json based config file `(config.json)` as given below:


```json
{
  "rnz": "publicBus",
  "account": "account.msa",
  "debug": "true"
}
```

This config file tells the mesh client to find more peers at `publicBus` rendezvous point, use `account.msa` account file and print all debug logs `("debug": "true")`.

A sample config file is avaiable [here](docs/config.json)

_______

## For data subscribers

You can quickly test out the platform by following the guide in this repo.

>Note that by following the above guide you won't be able to obtain realtime data and it should only be used to try out the platform.

To get realtime data you need to fire up `mesh` as a subscriber and connect via RPC.

**1. Running `mesh` as a subscriber node**

This can be done by running `mesh` with `en-sub` flag.
```
mesh -c docs/config.json --en-sub
```

This will do multiple things. First of all it will find other peers at `rnz` point and connect to them. It will then start an RPC server at default port `5555`.

>Note: If you want to use js client for mesh-gtfsr, you need to enable web RPC using `--web-rpc` flag. Without this you won't be able to connect to mesh RPC through a browser. Web RPC runs on a different port (Default: `5556`)

**2. Once `mesh` client is up and running we can connect with it via RPC**

Assuming we want to consume GTFS-Realtime data. Python client for `mesh-gtfsr` can be found here.

Install it with 
```
pip3 install mesh-gtfsr
```

Now we can start consuming GTFS-Realtime data for some specific *geospace*. `Delhi` in the given example. [Here](https://github.com/pravahio/go-mesh/wiki/Geospaces) you can find a list of all available *geospaces*.
```py
from mesh_gtfsr.mesh import MeshGTFSR

def main():
  m = MeshGTFSR()

  feed = m.subscribe([
    '/in/delhi'
  ])
  
  # You get feed in 'f'
  # and geospace in 'g
  for f, g in feed:
    print('Geospace: ' + g)
    print('Feed: ' + str(f)) 

if "__main__" == __name__:
  main()
```

`feed` is a tuple of `(feed:FeedMessage, geospace:string)`  as defined in [GTFS-Realtime specs](https://github.com/google/transit/blob/master/gtfs-realtime/proto/gtfs-realtime.proto)

If you want to get the data directly in the browser use the following guide.

_______

## For data publishers

**1. Running `mesh` as a publisher node**

This can be done by running `mesh` with `en-pub` flag.
```
mesh -c docs/config.json --en-pub
```

**2. Once `mesh` client is up and running we can connect to it via RPC and start publishing.**

Assuming we want to publish GTFS-Realtime data. Python client for `mesh-gtfsr` can be found here.

Install it with 
```
pip3 install mesh-gtfsr
```

Now we can start publishing GTFS-Realtime data.

```py
import time
from mesh_gtfsr.mesh import MeshGTFSR

GEOSPACE = [
  '/in/delhi'
]

def main():
  m = MeshGTFSR()

  rt = {
    "header": {
      "timestamp": time.time()
    },
    "entity": [{
      "id": "VEHICLE_ID",
      "vehicle": {
          "position": {
          "latitude": 77.23524,
          "longitude": 22.35343
          }
      }
    }]
  }

  m.registerToPublish(GEOSPACE)
  m.publish(GEOSPACE, rt)

if "__main__" == __name__:
  main()
```