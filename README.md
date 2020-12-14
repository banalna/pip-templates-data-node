# <img src="https://github.com/pip-services/pip-services/raw/master/design/Logo.png" alt="Pip.Services Logo" style="max-width:30%"> <br/> Beacons microservice

This is the Beacons microservice from the Pip.Templates library. 

The microservice currently supports the following deployment options:
* Deployment platforms: Standalone Process
* External APIs: HTTP/REST, gRPC
* Persistence: Memory, Flat Files, MongoDB, Couchbase

This microservice does not depend on other microservices.

<a name="links"></a> Quick Links:

* [Download Links](doc/Downloads.md)
* [Development Guide](doc/Development.md)
* [Configuration Guide](doc/Configuration.md)
* [Deployment Guide](doc/Deployment.md)
* Communication Protocols
  - [HTTP Version 1](swagger.yml)
  - [gRPC Version 1](src/protos/beacons_v1.proto)

## Contract

The logical contract of the microservice is presented below. 

```javascript
export class BeaconV1 implements IStringIdentifiable {
    public id: string;
    public site_id: string;
    public type?: string;
    public udi: string;
    public label?: string;
    public center?: any; // GeoJson
    public radius?: number;
}

// GeoJson Example:
// {
//     "type": "Point", 
//     "coordinates": [30, 10]
// }

export interface IBeaconsClientV1 {
    getBeacons(correlationId: string, filter: FilterParams, paging: PagingParams,
        callback: (err: any, page: DataPage<BeaconV1>) => void): void;

    getBeaconById(correlationId: string, beaconId: string,
        callback: (err: any, beacon: BeaconV1) => void): void;

    getBeaconByUdi(correlationId: string, udi: string,
        callback: (err: any, beacon: BeaconV1) => void): void;

    calculatePosition(correlationId: string, siteId: string, udis: string[], 
        callback: (err: any, position: any) => void): void;

    createBeacon(correlationId: string, beacon: BeaconV1,
        callback: (err: any, beacon: BeaconV1) => void): void;

    updateBeacon(correlationId: string, beacon: BeaconV1,
        callback: (err: any, beacon: BeaconV1) => void): void;

    deleteBeaconById(correlationId: string, beaconId: string,
        callback: (err: any, beacon: BeaconV1) => void): void;            
}

```

## Download

Right now, the only way to get the microservice is to check it out directly from the GitHub repository
```bash
git clone https://github.com/pip-templates/pip-templates-microservice-node.git
```

The Pip.Service team is working on implementing packaging, to make stable releases available as zip-downloadable archives.

## Run

Add the **config.yml** file to the config folder and set configuration parameters as needed.

For more information on microservice configuration, see [The Configuration Guide](Configuration.md).

The microservice can be started using the command:
```bash
node bin/run.js
```

## Use

The easiest way to work with the microservice is through the client SDK. 

If you use node.js, then get references to the required libraries:
- Pip.Services3.Commons : https://github.com/pip-services3-node/pip-services3-commons-node
- Pip.Services3.Rpc: 
https://github.com/pip-services3-node/pip-services3-rpc-node

<!-- Todo: rename pip-templates-microservice-node? -->
Add classes from the **pip-services3-commons-node** and **pip-templates-microservice-node** packages
```javascript
import { ConfigParams } from 'pip-services3-commons-node';
import { FilterParams } from 'pip-services3-commons-node';
import { PagingParams } from 'pip-services3-commons-node';

import { BeaconV1 } from 'pip-templates-microservice-node';
import { BeaconTypeV1 } from 'pip-templates-microservice-node';
import { BeaconsCommandableHttpClientV1 } from 'pip-templates-microservice-node';
```

Define client configuration parameters that match the configuration of the microservice's external API
```javascript
// Client configuration
var httpConfig = ConfigParams.fromTuples(
	"connection.protocol", "http",
	"connection.host", "localhost",
	"connection.port", 8080
);
```

Instantiate the client and open a connection to the microservice
```javascript
// Create the client instance
let client = new BeaconsCommandableHttpClientV1();

// Configure the client
client.configure(httpConfig);

// Connect to the microservice
client.open(null, (err) => {
  if (err) {
    // Error handling...
  }
});
    
// Work with the microservice
...
```

The client is now ready to perform operations
```javascript
// Define a beacon
let beacon: BeaconV1 = {
    id: '1',
    udi: '00001',
    type: BeaconTypeV1.AltBeacon,
    site_id: '1',
    label: 'TestBeacon',
    center: { type: 'Point', coordinates: [ 0, 0 ] },
    radius: 50
};

async.series([
    // Create the beacon
    (callback) => {
        this.client.createBeacon(
            null,
            BEACON1,
            (err, beacon) => {
                if(err){
                  // Error handling...
                }
                
                // Do something with the returned beacon...

                callback();
            }
        );
    },
    // Get a list of beacons
    (callback) => {
        this.client.getBeacons(
            null,
            FilterParams.fromTuples(
              "label", "TestBeacon",
            ),
            new PagingParams(0, 10),
            (err, page) => {
                if(err){
                  // Error handling...
                }

                // Do something with the returned page...
                // E.g. beacon = page.data[0];

                callback();
            }
        )
    }
], (err, results) => {
    if (err) {
        // Error handling...
    }
});
```

## Acknowledgements

This microservice was created and currently maintained by *Sergey Seroukhov*.
