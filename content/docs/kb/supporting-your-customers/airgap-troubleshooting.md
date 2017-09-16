+++
date = "2017-09-05T00:00:00Z"
lastmod = "2017-09-05T00:00:00Z"
title = "Airgap Troubleshooting"
weight = "999999"
categories = [ "Knowledgebase", "Supporting Your Customers" ]
+++

### An “airgapped” environment is a network that has no path to inbound or outbound internet traffic at all.  As a result, there are a few unique issues sometimes found during installation. 
### Below are some common errors and their solutions:

### 1. Internal Server Error
```bash
docker logs replicated-ui
WARNING 2016/08/24 19:56:27 airgap/airgap.go:144 Airgap package at /tmp/getelk.airgapllkk does not exist
ERROR 2016/08/24 19:56:27 premkit/log/gin.go:52 [GIN] 412 | 328.002094ms | | POST /v0.1/license/airgap
```
#### Solution: Determine whether path or airgap file are incorrect
To verify license is airgap-enabled:
```bash
docker logs replicated-ui
WARNING 2016/08/24 19:56:27 airgap/airgap.go:144 Airgap package at /tmp/getelk.airgapllkk does not exist
ERROR 2016/08/24 19:56:27 premkit/log/gin.go:52 [GIN] 412 | 328.002094ms | | POST /v0.1/license/airgap
```

### 2. No Disk Space
```bash
docker logs replicated
ERROR 2016/08/24 22:14:45 airgap/airgap.go:166 write /tmp/airgap-package492843268/images/public/elasticsearch:latest: no space left on device
ERROR 2016/08/24 22:14:45 premkit/log/gin.go:52 [GIN] 500 | 8.964383639s | | POST /v0.1/license/airgap
```
#### Solution: Manual Configuration to Expand Disk Space
Here we create the storage directory and then add a 100G (seek=100) sparse file for the storage pool.

```bash
service stop docker
rm -rf /var/lib/docker
mkdir -p /var/lib/docker/devicemapper/devicemapper
dd if=/dev/zero of=/var/lib/docker/devicemapper/devicemapper/data bs=1G count=0 seek=100
service start docker
```
It is also possible to use a device for the filesystem which would improve performance, we will have to wipe out "/var/lib/docker" as well so make sure to backup any containers. If it is a physical disk or an additional disk on the Cloud fdisk -l will be useful locating the disk we want to use.

```bash 
service stop docker
rm -rf /var/lib/docker
ln -s /dev/sdb /var/lib/docker/devicemapper/devicemapper/data
```
```bash
service start docker
```

### 3. Missing Intermediate Versions

```bash
docker logs replicated
ERROR 2016/08/26 20:55:05 airgap/airgap.go:224 No Major.Minor.Patch elements found
ERROR 2016/08/26 20:55:05 premkit/log/gin.go:52 [GIN] 500 | 33.939218035s | | POST /v0.1/license/airgap
```
#### Solution: Create a new license and Airgap package 
See [documentation](https://help.staging.replicated.com/docs/distributing-an-application/create-licenses/#airgap-download-enabled)


### 4. Restore Errors
Snapshot does not grab the remote images

#### Solution: Manually load these missing images from the airgap file.
1. Copy the .airgap file to the restore server

2. Untar the contents of the airgap file into a temporary folder
```bash
mkdir airgap
cd airgap
tar xfzv ../myairgap.airgap
```

3. Load each of the images into docker
```bash
find . -type f -exec docker load -i {} \;
```

4. Review the image names that were loaded
```bash
docker images
```

5. Re-name any images that are mis-named by re-tagging the image
```bash
docker tag <DOCKER-IMAGE-ID>  <DOCKER-IMAGE-NAME>:<DOCKER-IMAGE-TAG>
```

See [airgap documentation](https://help.replicated.com/docs/distributing-an-application/airgapped-installations/) for more information.