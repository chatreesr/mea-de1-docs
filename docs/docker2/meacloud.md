# MEA Cloud Environment

## การติดตั้ง Docker บน MEA Virtual Machines

ท่าจะเยอะกว่าลงเองบน Linux เครื่องส่วนตัวเพราะมีข้อจำกัดเรื่อง การกำหนด User ตอน Provision VM, Proxy และ Partition นิดหน่อย **อย่าเลือก Windows** และติดตั้ง Docker Desktop เพราะมีการคิด License สำหรับ Enterprise ที่มีขนาดพนักงานมากกว่า 250 คน งานอาจจะเข้าได้

### Request a VM

ขอ Virtual Machine จากระบบ [vRealize Automation](https://cld-vra01.mea.or.th/vcac/) ใช้ Username & Password จากระบบ AD โดย Spec ขั้นต่ำควรจะ

**Development Machine**

- OS: Ubuntu 18.04 LTS (Ubuntu 20.04 ไม่มีให้เลือก และ มีปัญหากับ Docker Engine)
- vCPU: 2
- RAM: 4 GB
- HDD1: 100 GB (Fixed)
- HDD2: 500 GB
- Network: ถ้ารันโปรแกรมที่ต้อง Service ภายนอกเลือกวง Customer Service ถ้าใช้งานเองเลือก Back Office
- ตั้งชื่อเริ่มต้นด้วย `DPD` เช่น `DPD-Docker-1`
- ใส่เหตุผลการขอไปด้วยว่า เอามาทำอะไร

### Update OS & Create User

หลังจาก Cloud Admin อนุมัติการใช้งาน VM แล้ว เช็ค Email จะได้ IP เครื่อง และ Username + Password สำหรับ Secure Shell

ทำการ Secure Shell เข้าเครื่องโดยใช้ PowerShell:

```bash
ssh root@ip_address
```

ทำการอัพเดท Repository และ อัพเกรดระบบ

```bash
apt update && apt upgrade -y
```

สร้าง Non-root user ด้วยคำสั่ง

```bash
adduser <username>
```

Add User เข้ากลุ่ม `sudo` เพื่อให้มีสิทธิ Administrator ผ่านคำสั่ง `sudo`

```bash
usermod -aG sudo <username>
```

### Setup SSH Key (Optional)

**ทำที่เครื่อง Laptop/PC** ไม่ใช่บน VM

โดยปกติการใช้ SSH Log in แบบใช้ Password ค่อนข้างจะไม่ Secure เพราะว่า Password ชอบหลุด วิธีนึงที่ใช้แก้คือใช้ SSH Passwordless Authentication ผ่าน Keys แทน ในตัวอย่างจะไม่ไป Disable password ออก เผื่อใช้ภายหลังแต่เครื่อง Production จริงๆ **ควรจะพิจารณาเอา Password ออก และ ใช้ Keys ทดแทน**

สร้าง SSH Key โดยใช้ Email เป็น Label ด้วยคำสั่ง (ใช้ Default ทั้งหมดเลยก็ได้)

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

(**Windows**) Copy SSH Key ไปที่เครื่อง VM

บางกรณี อาจจะต้อง Login ด้วย root เข้าไปสร้างไฟล์ `.ssh/authorized_keys` รอไว้ก่อน

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh <username>@<ip_address> "cat >> .ssh/authorized_keys"
```

(**Mac & Linux**) Copy SSH Key

Mac ลง Tool ก่อน ถ้าใช้ HomeBrew `brew install ssh-copy-id`

```bash
ssh-copy-id <username>@<ip_address>
```

ทดสอบด้วยคำสั่ง

```bash
ssh <username>@<ip_address>
```

เข้า SSH ได้เลยโดยไม่ต้องใส่ Password ขั้นตอนถัดไปคือ Disable Password ทิ้ง ถ้าพร้อมที่จะ Deploy Production 100% แล้ว[ทำตามนี้](https://stackoverflow.com/questions/20898384/disable-password-authentication-for-ssh)

**หลังจากเซ็ตค่าหมดแล้วให้ Restart ด้วยคำสั่ง `sudo reboot` และ Log In ด้วย `username` ที่สร้างในการทำขั้นตอนต่อไป (ไม่แนะนำให้ใช้ root)**

### Install Docker

ก่อนการติดตั้ง Docker ต้องเซ็ต Proxy สำหรับ curl ก่อน แก้ไขไฟล์ `~/.bashrc` โดยเพิ่ม

```bash
export http_proxy="http://proxy.mea.or.th:9090"
export https_proxy="http://proxy.mea.or.th:9090"
export no_proxy="localhost,172.16.0.0/16,172.17.0.0/16,192.169.254.0/24"
```

ติดตั้ง Package ที่จำเป็น

**สำคัญ** ไม่ควรลง Docker ผ่าน `snap` ตามคำแนะนำของ Ubuntu เพราะ Bug เยอะ และ Docker ไม่ได้ซัพพอร์ต ควรหลีกเลี่ยงเพราะปัญหาเยอะ

```bash
sudo apt update
```

```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

ดาวโหลด Docker's GPG key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

เพิ่ม Apt Repository สำหรับ Docker:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

ลง Docker

```bash
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io
```

ลง Docker Compose v2 สำหรับ User ตัวเอง:

```bash
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
mkdir -p $DOCKER_CONFIG/cli-plugins
curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
```

เพิ่ม Executable Permission:

```bash
chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose
```

### Add User to Group Docker

เพิ่ม User ตัวเองเข้า Group `docker` เพื่อให้ตอนใช้งานไม่ต้อง `sudo` ทุกคำสั่ง

```bash
sudo usermod -aG docker <username>
```

### Start Docker & Enable Startup Service on Boot

Start Docker daemon:

```bash
sudo systemctl start docker
```

ตั้งค่าให้ Docker รันอัตโนมัติตอน Reboot:

```bash
sudo systemctl enable docker
```

ให้ Reboot 1 ครั้ง และ ลองทดสอบด้วยคำสั่ง

```bash
docker version
```

```bash
docker compose version
```

ถึงจุดนี้แล้ว **จะยังไม่สามารถ Pull หรือ Run Image ได้** ต้องตั้งค่า MEA Proxy ก่อน

### Configure `docker0` Network

**สำคัญ** หลังจากลงเสร็จใหม่ๆ Docker จะ Detect Network ภายในเครื่องของเรา โดย Network `docker0` จะเริ่มจาก `172.17.0.0/16` ถ้า Network ชนกับ Subnet เครื่อง VM Docker จะเพิ่ม Subnet ให้อัตโนมัติเป็น `172.18.0.0/16` ตอนสร้าง VM จาก vRealize ถ้าเราเลือก VM ที่อยู่ในวง `172.16.0.0/16` Docker จะสร้าง `docker0` network ให้ที่ `172.17.0.0/16` ซึ่ง **ชนกับ** IP Address ของ Cloud กฟน. ทั้งวง **ทำให้ เราไม่สามารถเข้าถึง Service หรือ VM อื่นๆในวง 172.17.0.0/16**

ตรวจสอบ IP Address Space ของ Docker 0

ถ้า `docker0` อยู่ในวง `172.17.0.0/16` ให้คอนฟิกเพิ่ม ย้าย `docker0` ไปที่อื่น

```bash
ip addr
```

ย้าย Subnet ของ `docker0`

สร้างไฟล์ที่ `/etc/docker/daemon.json` และใส่

```json
{
  "default-address-pools": [{ "base": "172.26.0.0/16", "size": 24 }]
}
```

`172.26.0.0/16` ใส่อะไรก็ได้ที่ไม่ชนกับ Private IP ภายใน

แก้ไขเสร็จแล้ว ให้ Restart Docker Daemon:

```bash
sudo systemctl restart docker
```

### Configure Proxy for Docker Daemon

แก้ไขให้ Docker Daemon ใช้ MEA Proxy โดยไปสร้างไฟล์ที่ `/etc/systemd/system/docker.service.d/proxy.conf`

```bash
[Service]
Environment="HTTP_PROXY=http://proxy.mea.or.th:9090"
Environment="HTTPS_PROXY=http://proxy.mea.or.th:9090"
Environment="NO_PROXY=localhost,172.17.0.0/16,172.16.0.0/16,192.169.254.0/24"
```

เซฟให้เรียบร้อยและ Restart Docker Daemon ใหม่

```bash
sudo systemctl restart docker
```

ตรวจสอบสถานะ Docker daemon ให้เป็น Active ก่อนทำต่อ ด้วยคำสั่ง:

```bash
systemctl status docker
```

ทดสอบ Pull Image:

ถึงจุดนี้ **ควรจะ Pull Image ได้แล้ว แต่ Process ใน Container ยังจะไม่สามารถออก Internet ได้**

```bash
docker container run hello-world
```

### Configure Proxies for Container

เพื่อให้ Container สื่อสารผ่าน Proxy และ ออก Internet ได้ต้องคอนฟิกเพิ่มเติม โดยไปสร้างไฟล์ที่ `~/.docker/config.json` และใส่

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.mea.or.th:9090",
      "httpsProxy": "http://proxy.mea.or.th:9090",
      "noProxy": "localhost,172.17.0.0/16,172.16.0.0/16,192.169.254.0/24"
    }
  }
}
```

ทดสอบด้วยการ Run Container ทดสอบการออก Internet:

```bash
docker container run --rm ubuntu apt-get update
```

ถ้าสามารถออก Internet ได้จะเห็นการ Update Apt Repo ผ่านทุกรายการ

### Configure Docker Data Directory

**สำคัญ** HDD1 ของ VM ที่ได้มาจากระบบ vRealize มีขนาดที่ 100GB ซึ่งค่อนข้างน้อยสำหรับการจัดเก็บ Docker Images ควรจะย้าย Data Directory ให้ไปอยู่ HDD2 ซึ่ง VMWare จะ Mount ไว้ที่ `/vol1`

หยุดการทำงานของ Docker:

```bash
sudo systemctl stop docker
```

แก้ไขไฟล์ `/etc/docker/daemon.json` โดยเพิ่ม Key-Value เข้าไป

```json
"data-root": "/vol1/docker"
```

Copy Docker Data เดิมไปที่ใหม่

```bash
sudo rsync -aP /var/lib/docker /vol1/docker
```

Rename ชื่อ Directory เดิม

```bash
sudo mv /var/lib/docker /var/lib/docker.bak
```

Start Docker Daemon

```bash
sudo systemctl start docker
```

ถ้าทดสอบใช้งานได้ตามปกติแล้ว สามารถลบ `/var/lib/docker.bak` ได้

### Test & Play

ถึงจุดนี้ Docker ควรจะใช้งานได้ครบถ้วนแล้ว โดยมีวิธีการทดสอบ และ Troubleshoot พื้นฐานตามด้านล่าง

ทดสอบการ Pull Image

```bash
docker image pull nginx
```

ทดสอบให้ Container ออก Internet

```bash
docker container run --rm ubuntu apt-get update
```

ทดสอบเช็ค IP ของ Container (ไม่ควรจะซ้ำกับ IP วง Cloud)

```bash
docker container run --rm centos ip addr
```

ถ้าเซ็ตย้าย Subnet ตามข้างบน จะเห็น IP Address อยู่ใน Subnet `172.26.0.0/16`

## การขอเปิดพอร์ต

โดย Default เครื่อง VM จะไม่สามารถสื่อสารอะไรกับ VM/Service ภายใน Cloud ได้เลย ต้องขอเปิดพอร์ตติดต่อที่ คุณบีม ธีรวิทย์ กทข. ฝวท. พร้อมรายละเอียด

- Source
- Destination
- Port + Protocol (TCP/UDP)
- Direction (In, Out, In-Out)
