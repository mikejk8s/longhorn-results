

### Test Specs

Google Cloud
Specs:
3 masters 3 workers
worker stats:
n1-standard-4 (4 vCPUs, 15 GB memory)
(4) 375gb local SSDS in mdadm pool

Vsphere
Specs:
???

## Dbench Results (reverted to fi pngs in this repo after)

2. 1000gb 3 replicas

    Random Read/Write IOPS: 831/732. BW: 56.6MiB/s / 26.2MiB/s
    Average Latency (usec) Read/Write: 17064.21/22616.65
    Sequential Read/Write: 125MiB/s / 38.8MiB/s
    Mixed Random Read/Write IOPS: 579/192

3. Single Replica 1000GB

    Random Read/Write IOPS: 779/725. BW: 56.7MiB/s / 24.8MiB/s
    Average Latency (usec) Read/Write: 16947.07/18955.40
    Sequential Read/Write: 148MiB/s / 38.6MiB/s
    Mixed Random Read/Write IOPS: 559/186


## Setup

Set up nodes:
1. Install docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
2. cat /sys/module/scsi_mod/parameters/use_blk_mq
 -- Check for Y -- If the value of this file is Y, then multi-queue SCSI is already enabled on your imported image
3. nvme drives - nvme0n1 --> nvme0n4

setup disks -- https://cloud.google.com/compute/docs/disks/local-ssd
1. lsblk

2. wipe disks
	sudo mkfs.ext4 -F /dev/nvme0n1
  sudo mkfs.ext4 -F /dev/nvme0n2
  sudo mkfs.ext4 -F /dev/nvme0n3
  sudo mkfs.ext4 -F /dev/nvme0n4

3. Mount without write cache flushing
sudo mount -o discard,defaults,nobarrier /dev/nvme0n1 /mnt/disks/nvme0n1
sudo mount -o discard,defaults,nobarrier /dev/nvme0n2 /mnt/disks/nvme0n2
sudo mount -o discard,defaults,nobarrier /dev/nvme0n3 /mnt/disks/nvme0n3
sudo mount -o discard,defaults,nobarrier /dev/nvme0n4 /mnt/disks/nvme0n4

4. Create single pool of SSDs
sudo mdadm --create /dev/md0 --level=0 --raid-devices=4 /dev/nvme0n1 /dev/nvme0n2 /dev/nvme0n3 /dev/nvme0n4

5. Format pool
sudo mkfs.ext4 -F /dev/md0

6. Make pool mount dir
mkdir -p /mnt/disks/ssd-array

7. Mount pool to dir
mount /dev/md0 /mnt/disks/ssd-array

8. Allow read/write
chmod a+w /mnt/disks/ssd-array

9. Mount to /etc/fstab
echo UUID=`sudo blkid -s UUID -o value /dev/md0` /mnt/disks/ssd-array ext4 discard,defaults,nofail 0 2 | sudo tee -a /etc/fstab

10. To get results from bastians fio
kubectl port-forward service/longhorn-single-replica 8009:80
kubectl port-forward service/longhorn-three-replica 8009:80

11. Download results
wget --recursive http://127.0.0.1:8009/server/


#### ./generate-py is to generate html indexes seen in the dirs

```bash
.
├── config.yaml
├── dbench.yaml
├── generate-index.py
├── index.html
├── io_test_results
│   ├── gcp
│   │   ├── index.html
│   │   ├── longhorn-1-replica-1000gbpvc-1-readops_latency.png
│   │   ├── longhorn-1-replica-1000gbpvc-1-writeops_latency.png
│   │   ├── longhorn-1-replica-1000gbpvc-2-readops_latency.png
│   │   ├── longhorn-1-replica-1000gbpvc-2-writeops_latency.png
│   │   ├── longhorn-1-replica-1000gbpvc-3-readops_latency.png
│   │   ├── longhorn-1-replica-1000gbpvc-3-witeops_latency.png
│   │   ├── longhorn-1-replicas-1000gpvc-1
│   │   │   ├── benchmarks
│   │   │   │   ├── index.html
│   │   │   │   ├── randread-01.json
│   │   │   │   ├── randread-02.json
│   │   │   │   ├── randread-04.json
│   │   │   │   ├── randread-08.json
│   │   │   │   ├── randread-16.json
│   │   │   │   ├── randread-32.json
│   │   │   │   ├── randwrite-01.json
│   │   │   │   ├── randwrite-02.json
│   │   │   │   ├── randwrite-04.json
│   │   │   │   ├── randwrite-08.json
│   │   │   │   ├── randwrite-16.json
│   │   │   │   └── randwrite-32.json
│   │   │   └── index.html
│   │   ├── longhorn-1-replicas-1000gpvc-2
│   │   │   ├── benchmarks
│   │   │   │   ├── index.html
│   │   │   │   ├── randread-01.json
│   │   │   │   ├── randread-02.json
│   │   │   │   ├── randread-04.json
│   │   │   │   ├── randread-08.json
│   │   │   │   ├── randread-16.json
│   │   │   │   ├── randread-32.json
│   │   │   │   ├── randwrite-01.json
│   │   │   │   ├── randwrite-02.json
│   │   │   │   ├── randwrite-04.json
│   │   │   │   ├── randwrite-08.json
│   │   │   │   ├── randwrite-16.json
│   │   │   │   └── randwrite-32.json
│   │   │   └── index.html
│   │   ├── longhorn-1-replicas-1000gpvc-3
│   │   │   ├── benchmarks
│   │   │   │   ├── index.html
│   │   │   │   ├── randread-01.json
│   │   │   │   ├── randread-02.json
│   │   │   │   ├── randread-04.json
│   │   │   │   ├── randread-08.json
│   │   │   │   ├── randread-16.json
│   │   │   │   ├── randread-32.json
│   │   │   │   ├── randwrite-01.json
│   │   │   │   ├── randwrite-02.json
│   │   │   │   ├── randwrite-04.json
│   │   │   │   ├── randwrite-08.json
│   │   │   │   ├── randwrite-16.json
│   │   │   │   └── randwrite-32.json
│   │   │   └── index.html
│   │   ├── longhorn-3-replicas-200gbpvc-1-readiops_latency.png
│   │   ├── longhorn-3-replicas-200gbpvc-1-writeops_latency.png
│   │   ├── longhorn-3-replicas-200gbpvc-2-readops-latency.png
│   │   ├── longhorn-3-replicas-200gbpvc-2-writeops-latency.png
│   │   ├── longhorn-3-replicas-200gbpvc-3-readops-latency.png
│   │   ├── longhorn-3-replicas-200gbpvc-3-writeops-latency.png
│   │   ├── longhorn-3-replicas-200gpvc-1
│   │   │   ├── benchmarks
│   │   │   │   ├── index.html
│   │   │   │   ├── randread-01.json
│   │   │   │   ├── randread-02.json
│   │   │   │   ├── randread-04.json
│   │   │   │   ├── randread-08.json
│   │   │   │   ├── randread-16.json
│   │   │   │   ├── randread-32.json
│   │   │   │   ├── randwrite-01.json
│   │   │   │   ├── randwrite-02.json
│   │   │   │   ├── randwrite-04.json
│   │   │   │   ├── randwrite-08.json
│   │   │   │   ├── randwrite-16.json
│   │   │   │   └── randwrite-32.json
│   │   │   └── index.html
│   │   ├── longhorn-3-replicas-200gpvc-2
│   │   │   ├── benchmarks
│   │   │   │   ├── index.html
│   │   │   │   ├── randread-01.json
│   │   │   │   ├── randread-02.json
│   │   │   │   ├── randread-04.json
│   │   │   │   ├── randread-08.json
│   │   │   │   ├── randread-16.json
│   │   │   │   ├── randread-32.json
│   │   │   │   ├── randwrite-01.json
│   │   │   │   ├── randwrite-02.json
│   │   │   │   ├── randwrite-04.json
│   │   │   │   ├── randwrite-08.json
│   │   │   │   ├── randwrite-16.json
│   │   │   │   └── randwrite-32.json
│   │   │   └── index.html
│   │   └── longhorn-3-replicas-200gpvc-3
│   │       ├── benchmarks
│   │       │   ├── index.html
│   │       │   ├── randread-01.json
│   │       │   ├── randread-02.json
│   │       │   ├── randread-04.json
│   │       │   ├── randread-08.json
│   │       │   ├── randread-16.json
│   │       │   ├── randread-32.json
│   │       │   ├── randwrite-01.json
│   │       │   ├── randwrite-02.json
│   │       │   ├── randwrite-04.json
│   │       │   ├── randwrite-08.json
│   │       │   ├── randwrite-16.json
│   │       │   └── randwrite-32.json
│   │       └── index.html
│   ├── index.html
│   ├── readme.md
│   └── vsphere
│       ├── generate-index.py
│       ├── index.html
│       ├── io\ read\ ceph.png
│       ├── io\ read\ hostpath.png
│       ├── io\ read\ longhorn\ default.png
│       ├── io\ read\ longhorn\ external\ disk.png
│       ├── io\ read\ portworx.png
│       ├── io\ read\ storageos.png
│       ├── io\ write\ ceph.png
│       ├── io\ write\ hostpath.png
│       ├── io\ write\ longhorn\ default.png
│       ├── io\ write\ longhorn\ external\ disk.png
│       ├── io\ write\ portworx.png
│       └── io\ write\ storageos.png
├── longhorn-1-replica.yaml
├── longhorn-3-replica.yaml
├── readme.md
├── scratch
│   ├── az-benchmarking-node.json
│   ├── az-master-node.json
│   └── index.html
├── storageclass-1-replica.yaml
└── storageclass-3-replicas.yaml
```