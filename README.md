# Docker Beginner Command List

> ## 1. Docker version
```bash
docker version
```
<br>

> ## 2. Docker Image
```bash
# List docker image:
docker image ls

# Pull/download image:
docker image pull <image_name>:<tag>

# DELETE/REMOVE image:
docker image rm <image_name>:<tag>
```
<br>


> ## 3. Docker Container
```bash
# List running container:
docker container ls

# List all container:
docker container ls -a


# CREATE container:
docker container create --name <container_name> <image_name>:<tag>

# START container:
docker container start <container_id|container_name>

# STOP container:
docker container stop <container_id|container_name>

# DELETE/REMOVE container:
docker container rm <container_id|container_name>

# Log container (detach mode):
docker container logs <container_id|container_name>

# Log container:
docker container logs -f <container_id|container_name>

# Masuk container:
## -i = interactive, menjaga input tetap aktif
## -t = alokasi pseudo-TTY (terminal akses)
docker container exec -i -t <container_id|container_name> /bin/bash
```
<br>


> ## 4. Port Fowarding
```bash
# Port fowarding:
docker container create --name <container_name> --publish <host_port>:<container_port> <image_name>:<tag> 
```
```bash
# Contoh:
docker container create --name myfirstcontainer --publish 8080:80 nginx:latest
```
<br>


> ## 5. SET Environment Variable
```bash
# SET Environment Variable
docker container create --name <container_name> --env KEY1="value1" --env KEY2="value2" <image_name>:<tag>
```
```bash
# Contoh
docker container create --name contohmongo --publish 32041:27017 \
--env MONGO_INITDB_ROOT_USERNAME=admin \
--env MONGO_INITDB_ROOT_PASSWORD=admin \
mongo:latest
```
<br>


> ## 6. Monitoring Container Computation Usage (CPU/GPU/RAM):
```bash
docker container stats
```
<br>


> ## 7.  Limit Container Resource:
> You can limit a container from using resources by using:<br>
> - --memory <limit[b|k|m|g]> <br>
> - --cpus <limit>
> 
> Memory limit details:
> - b = bytes
> - k = kilobytes
> - m = megabytes
> - g = gigabytes

```bash
docker container create --name <container_name> --memory <limit[b|k|m|g]> --cpus <limit> --publish <host_port>:<container_port> <image_name>:<tag>
```
```bash
# Contoh:
docker container create --name container_smallnginx --memory 100m --cpus 0.5 --publish 8081:80 nginx:latest
```
<br>


> ## 8. Bind Mount
> Mounting/sharing file atau folder dari container-host | host-container<br>
> Jika file tidak ada, secara automatis buat folder dengan menggunakan: "--mount" atau "-v"
>
> Terdapat beberapa parameter:
>
> |Parameter   |Keterangan                         |
> |------------|-----------------------------------|
> |type        |bind or volume                     |
> |source      |Lokasi file host/sistem            |
> |destination |Lokasi file container              |
> |readonly    |File hanya bisa dibaca di container|

```bash
docker container create \
--name <container_name> \
--mount "type=bind, \
         source=<host_path>, \
         destination=<container_image_path>" \
...
```

```bash
# Contoh:
docker container create \
--name mongodata \
--mount "type=bind,source=C:\Users\victo\pdf_encrypt_decrypt,destination=/data/db" \
--publish 27019:27017 \
--env MONGO_INITDB_ROOT_USERNAME=eko \
--env MONGO_INITDB_ROOT_PASSWORD=eko \
mongo:latest
```
<br>

> ## 9. Docker Volume
> - Mirip dengan Bind Mounts, bedanya adalah terdapat management Volume, dimana kita bisa membuat Volume, melihat daftar Volume, dan menghapus Volume
>
> - Bisa dianggap storage yang digunakan untuk menyimpan data, bedanya dengan Bind Mounts, data di manage oleh Docker
```bash
# List Docker Volume:
docker volume ls
```
<br>

> ## 10. Container Volume
> Membuat container volume berurutan:
> 1. CREATE Volume
> 2. CREATE Container dan mountin: --mount "type=volume,source=<volume_name>"
> 3. START Container
```bash
# 1. CREATE Volume:
docker volume create --name <volume_name>

# 2. CREATE Container, mount type=volme,source=<container_name>:
docker container create --name <container_name> --publish <host_port>:<container_port> --mount "type=volume,source=<volume_name>,destination=<container_image_path>" <image_name>:<tag>

# 3. START Container
docker container start <container_name>
```

```bash
# Contoh:
# 1. CREATE Volume:
docker volume create --name mongodata

# 2. CREATE Container, mount type=volme,source=<container_name>:
docker container create --name mongovolume --publish 27019:27017 --mount "type=volume,source=mognovolume,destination=/data/db" --env MONGO_INITDB_ROOT_USERNAME=eko --env MONGO_INITDB_ROOT_PASSWORD=eko mongo:latest

# 3. START Container
docker container start mongovolume
```
<br>


> ## 11. Backup Volume
> Backup data container yang menggunakan volume ke host
> 1. Harus matikan Container yang mengunakan VOLUME tersebut
> 2. Create a new *temporary* CONTAINER for *backup* dengan 2 mount<br>
**FIRST**, VOLUME yang mau di *backup* dimasukkan ke *temporary* Container tersebut.<br>
**SECOND**, buat bind mount folder yang ada di host.<br>
**THIRD**, (HARUS MASUK KE *TEMPORARY* CONTAINER TERSEBUT) dan lakukan *backup* <br>
**LAST**, you can delete the *TEMPORARY* CONTAINER
```bash
mkdir mybackup

docker container create \
--name temp_container_for_backup \
--mount "type=bind,source=<C:\Users\victo\[Internship] Data Labs\mybackup>,destination=/backup" \
--mount "type=volume,source=<nama_volume_yang_mau_dibackup>,destination=/data" \
nginx:latest

docker container start temp_container_for_backup
docker container exec -i -t temp_container_for_backup /bin/bash
tar cvf zipbackup.tar.gz /data
docker container stop temp_container_for_backup
docker container rm temp_container_for_backup
```

> Perintah yang lebih singkat
```bash
docker container run --rm --name temp_container_for_backup \
--mount "type=bind,source=<C:\Users\victo\[Internship] Data Labs\mybackup>,destination=/backup" \
--mount "type=volume,source=<nama_volume_yang_mau_dibackup>,destination=/data" \
nginx:latest \
tar cvf another-backup.tar.gz /data
```
<br>


> ## 12. Restore Volume
> Untuk restore volume, mirip dengan backup, prosesnya hanya kebalik. Tetap perlu membuat temporary container.<br>
> yang berubah hanya command untuk extract/restore.
```bash
tar xvf filename.tar.gz --strip 1
```
```bash
docker container run --rm --name temp_container_for_restore \
--mount "type=bind,source=<C:\Users\victo\[Internship] Data Labs\mybackup>,destination=/backup" \
--mount "type=volume,source=<new_volume_name>,destination=/data" \
nginx:latest \
bash -c "cd /data && tar xvf filename.tar.gz --strip 1"
```
<br>


> ## 13. Docker Network
>  Setiap Container terisolasi. Menggunakan Docker Network, Container dapat saling berkomunikasi.<br>
> Jenis Network Driver:<br>
> 1. bridge (network virtual, Container yang terkoneksi di bridge network, dapat saling berkomunikasi)
> 2. host (hanya bisa di OS Linux)
> 3. none (DEFAULT, driver untuk membuat network tidak dapat berkomunikasi)

```bash
docker network ls
```
```bash
# 1. CREATE Network:
docker network create --driver <jenisdriver> <networkname>

# 2. CREATE Network:
docker network rm <networkname>
```
<br>


> ## 14. Container Network
> Terlalu panjang..


> ## 15. Inpect
> Melihat lebih detil dari perintah ```ls```
```bash
docker image inspect <name>
docker container inspect <name>
docker volume inspect <name>
docker network inspect <name>
```
<br>

> ## 16. Prune
> Menghapus/membersihkan hal-hal yang sudah tidak terpakai di Docker yang sedang tidak jalan atau stop.
```bash
docker image prune
docker container prune
docker volume prune
docker network prune
docker system prune
```
