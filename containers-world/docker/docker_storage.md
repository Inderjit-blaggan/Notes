# 📦 Docker Storage & Volume Management (Advanced)

## 📁 Default Docker Storage Path

- Docker uses the following default directory to store container data:
```bash
/var/lib/docker
```

This location contains:

- `containers/`: container-specific data
- `volumes/`: managed volumes
- `image/`: image layers
- `overlay2/`: layer and diff information (if using overlay2 driver)

More details: [Docker Storage Overview](https://docs.docker.com/storage/)

---

## 🔗 Volume Mount vs Bind Mount

| Feature            | Bind Mount                                      | Volume Mount                        |
|--------------------|--------------------------------------------------|-------------------------------------|
| Path control       | You define exact host path                       | Docker manages the path             |
| Use case           | Share specific host folders                      | Persistent and portable storage     |
| Setup (Old Style)  | `docker run -v /host:/container`                | `docker run -v volname:/container`  |
| Setup (New Style)  | `--mount type=bind,source=/src,target=/target`  | `--mount type=volume,...`           |

---

### ✅ Examples

#### Bind Mount (Old style)

```bash
docker run -v /data/mysql:/var/lib/mysql mysql
```

#### Bind Mount (New style)

```bash
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

#### Named Volume

```bash
docker volume create dbdata
docker run --mount source=dbdata,target=/var/lib/mysql mysql
```

---

## 🧰 Storage Drivers

Docker uses storage drivers to manage how container layers are stored.

### 🔍 Check Current Storage Driver:

```bash
docker info | grep "Storage Driver"
```

### 📚 Supported Storage Drivers:

| Driver        | Description                                  | OS Compatibility          |
|---------------|----------------------------------------------|---------------------------|
| `overlay2`    | Preferred modern driver                      | Linux (Kernel ≥ 4.x)      |
| `aufs`        | Used in older systems                        | Legacy Ubuntu             |
| `btrfs`       | Supports snapshotting                        | Experimental              |
| `devicemapper`| Block-level thin provisioning (CentOS)       | CentOS, RHEL              |
| `zfs`         | Advanced filesystem features (snapshotting)  | ZFS-supported systems     |

> More info: [Storage Driver Comparison](https://docs.docker.com/storage/storagedriver/)

---

## 📘 Further Reading

- [Docker Storage Official Docs](https://docs.docker.com/storage/)
- [Volume Concepts](https://docs.docker.com/storage/volumes/)
- [Bind Mounts](https://docs.docker.com/storage/bind-mounts/)
- [Storage Drivers Explained](https://docs.docker.com/storage/storagedriver/)

