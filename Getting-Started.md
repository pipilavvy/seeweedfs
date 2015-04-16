# Getting Started

## Installing SeaweedFS

Download a proper version from [SeaweedFS download page][bintray].

[bintray]: https://bintray.com/chrislusf/Weed-FS/seaweed

Decompress the downloaded file. You will only find one executable file, either "weed" on most systems or "weed.exe" on windows.

Put the file "weed" to all related computers, in any folder you want. Use 

```bash
./weed -h # to check available options
```

### Set up Weed Master

```bash
./weed master -h # to check available options
```

If no replication is required, this will be enough. The "mdir" option is to configure a folder where the generated sequence file ids are saved.

```bash
./weed master -mdir="."
```

If you need replication, you would also set the configuration file. By default it is "/etc/weedfs/weedfs.conf" file. The example can be found in RackDataCenterAwareReplication

### Set up Weed Volume Server

```bash
./weed volume -h # to check available options
```

Usually volume servers are spread on different computers. They can have different disk space, or even different operating system.

Usually you would need to specify the available disk space, the Weed Master location, and the storage folder.

```bash
./weed volume -max=100 -mserver="localhost:9333" -dir="./data"
```

### Cheat Sheet: Setup One Master Server and One Volume Server

Actually, forget about previous commands. You can setup one master server and one volume server in one shot:

```bash
./weed server -dir="./data"
# same, just specifying the default values
# use "weed server -h" to find out more
./weed server -master.port=9333 -volume.port=8080 -dir="./data"
```

## Testing SeaweedFS

With the master and volume server up, now what? Let's pump in a lot of files into the system!

```bash
./weed upload -dir="/some/big/folder"
```

This command would recursively upload all files. Or you can specify what files you want to include.

```bash
./weed upload -dir="/some/big/folder" -include=*.txt
```

Then, you can simply check "du -m -s /some/big/folder" to see the actual disk usage by OS, and compare it with the file size under "/data". Usually if you are uploading a lot of textual files, the consumed disk size would be much smaller since textual files are gzipped automatically.

Now you can use your tools to hit SeaweedFS as hard as you can.

## Using SeaweedFS in docker

You can use image "cydev/weed" or build your own with [dockerfile][] in the root of repo.

[dockerfile]: https://github.com/chrislusf/seaweedfs/blob/master/Dockerfile

### Using pre-built Docker image

```bash
docker run --name weed cydev/weed server
```

And in another terminal

```bash
IP=$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' weed)
curl "http://$IP:9333/cluster/status?pretty=y"	
{
  "IsLeader": true,
  "Leader": "localhost:9333"
}
# use $IP as host for api queries
```

### Building image from dockerfile

Make a local copy of seaweedfs from github

```bash
git clone https://github.com/chrislusf/seaweedfs.git
```

Minimal Image (~19.6 MB)

```bash
docker build --no-cache -t 'cydev/weed' .
```

Go-Build Docker Image (~764 MB)

```bash
mv Dockerfile Dockerfile.minimal
mv Dockerfile.go_build Dockerfile
docker build --no-cache -t 'cydev/weed' .
```

### In production

To gain persistency you can use docker volumes.

```bash
# start our weed server daemonized
docker run --name weed -d -p 9333:9333 -p 8080:8080 \
  -v /opt/weedfs/data:/data cydev/weed server -dir="/data" \ 
  -publicIp="$(curl -s cydev.ru/ip)"
```

Now our SeaweedFS server will be persistent and accessible by localhost:9333 and :8080 on host machine.
Dont forget to specify "-publicIp" for correct connectivity.