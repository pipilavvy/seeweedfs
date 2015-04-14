## Master server

You can append to any HTTP API with &pretty=y to see a formatted json output.

### Assign a file key
This operation is very cheap. Just increase a number in master server's memory.

```bash
# Basic Usage:
curl http://localhost:9333/dir/assign
{"count":1,"fid":"3,01637037d6","url":"127.0.0.1:8080",
 "publicUrl":"localhost:8080"}
# To assign with a specific replication type:
curl "http://localhost:9333/dir/assign?replication=001"
# To specify how many file ids to reserve
curl "http://localhost:9333/dir/assign?count=5"
# To assign a specific data center
curl "http://localhost:9333/dir/assign?dataCenter=dc1"
```

### Lookup volume

We would need to find out whether the volumes have moved.

```bash
curl "http://localhost:9333/dir/lookup?volumeId=3&pretty=y"
{
  "locations": [
    {
      "publicUrl": "localhost:8080",
      "url": "localhost:8080"
    }
  ]
}
# Other usages:
# You can actually use the file id to lookup, if you are lazy to parse the file id.
curl "http://localhost:9333/dir/lookup?volumeId=3,01637037d6"
# If you know the collection, specify it since it will be a little faster
curl "http://localhost:9333/dir/lookup?volumeId=3&collection=turbo"
```

### Force garbage collection

If your system has many deletions, the deleted file's disk space will not be synchronously re-claimed. There is a background job to check volume disk usage. If empty space is more than the threshold, default to 0.3, the vacuum job will make the volume readonly, create a new volume with only existing files, and switch on the new volume. If you are impatient or doing some testing, vacuum the unused spaces this way.

```bash
curl "http://localhost:9333/vol/vacuum"
curl "http://localhost:9333/vol/vacuum?garbageThreshold=0.4"
```

The garbageThreshold=0.4 is optional, and will not change the default threshold. You can start volume master with a different default garbageThreshold.

This operation is not trivial. It will try to make a copy of the .dat and .idx files, skipping deleted files, and switch to the new files, removing the old files.

### Pre-Allocate Volumes

One volume servers one write a time. If you need to increase concurrency, you can pre-allocate lots of volumes.

```bash
curl "http://localhost:9333/vol/grow?replication=000&count=4"
{"count":4}
# specify a collection
curl "http://localhost:9333/vol/grow?collection=turbo&count=4"
# specify data center
curl "http://localhost:9333/vol/grow?dataCenter=dc1&count=4"
```

This generates 4 empty volumes.

### Check System Status

```bash
curl "http://10.0.2.15:9333/cluster/status?pretty=y"
{
  "IsLeader": true,
  "Leader": "10.0.2.15:9333",
  "Peers": [
    "10.0.2.15:9334",
    "10.0.2.15:9335"
  ]
}
curl "http://localhost:9333/dir/status?pretty=y"
{
  "Topology": {
    "DataCenters": [
      {
        "Free": 3,
        "Id": "dc1",
        "Max": 7,
        "Racks": [
          {
            "DataNodes": [
              {
                "Free": 3,
                "Max": 7,
                "PublicUrl": "localhost:8080",
                "Url": "localhost:8080",
                "Volumes": 4
              }
            ],
            "Free": 3,
            "Id": "DefaultRack",
            "Max": 7
          }
        ]
      },
      {
        "Free": 21,
        "Id": "dc3",
        "Max": 21,
        "Racks": [
          {
            "DataNodes": [
              {
                "Free": 7,
                "Max": 7,
                "PublicUrl": "localhost:8081",
                "Url": "localhost:8081",
                "Volumes": 0
              }
            ],
            "Free": 7,
            "Id": "rack1",
            "Max": 7
          },
          {
            "DataNodes": [
              {
                "Free": 7,
                "Max": 7,
                "PublicUrl": "localhost:8082",
                "Url": "localhost:8082",
                "Volumes": 0
              },
              {
                "Free": 7,
                "Max": 7,
                "PublicUrl": "localhost:8083",
                "Url": "localhost:8083",
                "Volumes": 0
              }
            ],
            "Free": 14,
            "Id": "DefaultRack",
            "Max": 14
          }
        ]
      }
    ],
    "Free": 24,
    "Max": 28,
    "layouts": [
      {
        "collection": "",
        "replication": "000",
        "writables": [
          1,
          2,
          3,
          4
        ]
      }
    ]
  },
  "Version": "0.47"
}
```

## Volume Server

### Upload File

```bash
curl -F file=@/home/chris/myphoto.jpg http://127.0.0.1:8080/3,01637037d6
{"size": 43234}
```

The size returned is the size stored on SeaweedFS, sometimes the file is automatically gzipped based on the mime type.

### Upload File Directly

```bash
curl -F file=@/home/chris/myphoto.jpg http://localhost:9333/submit
{"fid":"3,01fbe0dc6f1f38","fileName":"myphoto.jpg","fileUrl":"localhost:8080/3,01fbe0dc6f1f38","size":68231}
```

This API is just for convenience. The master server would get an file id and store the file to the right volume server.
It is a convenient API and does not support different parameters when assigning file id. (or you can add the support and send a push request.)

### Delete File

```bash
curl -X DELETE http://127.0.0.1:8080/3,01637037d6
```
	
### Create а specific volume on a specific volume server

```bash
curl "http://localhost:8080/admin/assign_volume?replication=000&volume=3"
```

This generates volume 3 on this volume server.

If you use other replicationType, e.g. 001, you would need to do the same on other volume servers to create the mirroring volumes.

### Check Volume Server Status

```bash
curl "http://localhost:8080/status?pretty=y"
{
  "Version": "0.34",
  "Volumes": [
    {
      "Id": 1,
      "Size": 1319688,
      "RepType": "000",
      "Version": 2,
      "FileCount": 276,
      "DeleteCount": 0,
      "DeletedByteCount": 0,
      "ReadOnly": false
    },
    {
      "Id": 2,
      "Size": 1040962,
      "RepType": "000",
      "Version": 2,
      "FileCount": 291,
      "DeleteCount": 0,
      "DeletedByteCount": 0,
      "ReadOnly": false
    },
    {
      "Id": 3,
      "Size": 1486334,
      "RepType": "000",
      "Version": 2,
      "FileCount": 301,
      "DeleteCount": 2,
      "DeletedByteCount": 0,
      "ReadOnly": false
    },
    {
      "Id": 4,
      "Size": 8953592,
      "RepType": "000",
      "Version": 2,
      "FileCount": 320,
      "DeleteCount": 2,
      "DeletedByteCount": 0,
      "ReadOnly": false
    },
    {
      "Id": 5,
      "Size": 70815851,
      "RepType": "000",
      "Version": 2,
      "FileCount": 309,
      "DeleteCount": 1,
      "DeletedByteCount": 0,
      "ReadOnly": false
    },
    {
      "Id": 6,
      "Size": 1483131,
      "RepType": "000",
      "Version": 2,
      "FileCount": 301,
      "DeleteCount": 1,
      "DeletedByteCount": 0,
      "ReadOnly": false
    },
    {
      "Id": 7,
      "Size": 46797832,
      "RepType": "000",
      "Version": 2,
      "FileCount": 292,
      "DeleteCount": 0,
      "DeletedByteCount": 0,
      "ReadOnly": false
    }
  ]
}
```

## Filer server

### POST/PUT/Get files
```bash
# Basic Usage:
	//create or overwrite the file, the directories /path/to will be automatically created
	POST /path/to/file
	//get the file content
	GET /path/to/file
	//create or overwrite the file, the filename in the multipart request will be used
	POST /path/to/
	//return a json format subdirectory and files listing
	GET /path/to/
```
Examples:
```bash
# Basic Usage:
> curl -F file=@report.js "http://localhost:8888/javascript/"
{"name":"report.js","size":866}
> curl  "http://localhost:8888/javascript/report.js"   # get the file content
...
> curl -F file=@report.js "http://localhost:8888/javascript/new_name.js"    # upload the file with a different name
{"name":"report.js","size":866}
> curl  "http://localhost:8888/javascript/?pretty=y"            # list all files under /javascript/
{
  "Directory": "/javascript/",
  "Files": [
    {
      "name": "new_name.js",
      "fid": "3,034389657e"
    },
    {
      "name": "report.js",
      "fid": "7,0254f1f3fd"
    }
  ],
  "Subdirectories": null
}
```

### List files under a directory
This is for embedded filer only.

Some folder can be very large. To efficiently list files, we use a non-traditional way to iterate files. Every pagination you provide a "lastFileName", and a "limit=x". The filer locate the "lastFileName" in O(log(n)) time, and retrieve the next x files.

```bash
curl  "http://localhost:8888/javascript/?pretty=y&lastFileName=new_name.js&limit=2"
{
  "Directory": "/javascript/",
  "Files": [
    {
      "name": "report.js",
      "fid": "7,0254f1f3fd"
    }
  ]
}

```

### Move a directory
This is for embedded filer only.

Rename a folder is an O(1) operation, even for folders with lots of files.

```bash
# Change folder name from /javascript to /assets
> curl  "http://localhost:8888/admin/mv?from=/javascript&to=/assets"

# no files under /javascript now
> curl  "http://localhost:8888/javascript/?pretty=y"
{
  "Directory": "/javascript/",
  "Files": null,
  "Subdirectories": null
}

# files are moved to /assets folder
> curl  "http://localhost:8888/assets/?pretty=y"
{
  "Directory": "/assets/",
  "Files": [
    {
      "name": "new_name.js",
      "fid": "3,034389657e"
    },
    {
      "name": "report.js",
      "fid": "7,0254f1f3fd"
    }
  ],
  "Subdirectories": null
}
```

# Delete a file
```bash
> curl -X DELETE "http://localhost:8888/assets/report.js"
```
