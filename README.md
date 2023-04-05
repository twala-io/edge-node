# Edge Node

**Edge is currently under heavy development. An optimized version will be released soon**

## Goal/Purpose
Dedicated light node to upload and retrieve their CIDs. To do this, we decoupled the upload and retrieval aspect from the Estuary API node so we can create a node that can live on the "edge" closer to the customer.

By decoupling this to a light node, we achieve the following:
- dedicated node assignment for each customer. The customer or user can now launch an edge node and use it for both uploading to Estuary and retrieval using the same API keys issued from Estuary.
- switches the upload protocol. The user still needs to upload via HTTP but the edge node will transfer the file over to a delta node to make deals.

![image](https://user-images.githubusercontent.com/4479171/227985970-58bfead8-0906-4f2e-b7ae-b314508ee3e5.png)

## Features
- Only supports online/e2e verified deals for now.
- Accepts concurrent uploads (small to large)
- Stores the CID and content on the local blockstore using whypfs
- Save the data on local sqlite DB
- retries the storage deals if it fails. Uses delta `auto_retry` feature.
- periodically checks the status of the deals and update the database.
- For 32GB and above, the node will split the file into 32GB chunks and make deals for each chunk and car them. **[WIP]**

# Build
## `go build`
```
go build -tags netgo -ldflags '-s -w' -o edge
```

## `make`
```
make all
```

# Running
## Create the `.env` file
```
DB_NAME=edge-urdb
DELTA_NODE_API=https://cake.delta.store
DEAL_CHECK=600 # checks all deal status every 10mins.
```

## Running the daemon
```
./edge daemon --repo=/tmp/blockstore
```


# Gateway
This node comes with it's own gateway to serve directories and files.

View the gateway using:
- http://localhost:1313/gw/:cid
- http://localhost:1313/gw/ipfs/:cid

## Gateway home page
```
http://localhost:1313/
```
![image](https://user-images.githubusercontent.com/4479171/230234478-80f27572-6615-4dde-8507-39701acdd9ee.png)

## Dir Viewer
![image](https://user-images.githubusercontent.com/4479171/230234667-9e910d07-13bc-47b2-9662-0b3370981680.png)

# Pin and make a storage deal for your file(s) on Estuary

## Get API key
```
curl --location --request GET 'https://auth.estuary.tech/register-new-token'
{
    "expires": "2123-02-03T21:12:15.632368998Z",
    "token": "<API_KEY>"
}
```

## Upload and make a storage deal
```
curl --location --request POST 'http://localhost:1313/api/v1/content/add' \
--header 'Authorization: Bearer [API_KEY]' \
--form 'data=@"/path/to/file"'
{
    "status": "success",
    "message": "File uploaded and pinned successfully. Please take note of the id.",
    "id": 5,
    "cid": "bafybeicgdjdvwes3e5aaicqljrlv6hpdfsducknrjvsq66d4gsvepolk6y"
}
```

## View / download your file
```
http://localhost:1313/gw/bafybeicgdjdvwes3e5aaicqljrlv6hpdfsducknrjvsq66d4gsvepolk6y
http://localhost:1313/gw/ipfs/bafybeicgdjdvwes3e5aaicqljrlv6hpdfsducknrjvsq66d4gsvepolk6y
```

## Check the status of your content
This will return the status of the file(s) or cid(s) on edge. It'll also return the delta content_id.
```
curl --location --request GET 'http://localhost:1313/api/v1/status/5' \
--header 'Authorization: Bearer [API_KEY]'
{
    "content": {
        "ID": 5,
        "name": "aqua-plugin-231.8109.147.zip",
        "size": 1157548,
        "cid": "bafybeicgdjdvwes3e5aaicqljrlv6hpdfsducknrjvsq66d4gsvepolk6y",
        "delta_content_id": 2705,
        "status": "transfer-finished",
        "last_message": "transfer-finished",
        "miner": "",
        "created_at": "2023-04-05T09:08:11.839358-04:00",
        "updated_at": "2023-04-05T09:09:00.105453-04:00"
    }
}
```

# Author
Protocol Labs Outercore Engineering
