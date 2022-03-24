# Docker Swarm

Built in container orchestration

## Swarm Architecture

## Swarm Init

เริ่มใช้งาน Docker Swarm ได้ด้วยคำสั่ง

```bash
docker swarm init
```

จะเห็น Output

```text
Swarm initialized: current node (w5rj9pqmcv0z2b052f8dr0qd2) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-6aq8xeo72o9kj5orlbzglt66fpxtw1dlmkb9djkgsatmpw8ugi-2jgc0rqmq2b5aet51dl89qbbz 192.168.1.77:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

สิ่งสำคัญคือ

- `docker swarm init` คือการสร้าง Manager Node แรกของ Swarm
- เราสามารถใช้คำสั่งตาม Output เพื่อเพิ่ม Manager หรือเพิ่ม Worker ได้
- หลังจาก Init แล้วเราจะสามารถใช้คำสั่ง `docker service <sub-command>` ในการรัน Container

## Swarm Nodes

ตรวจสอบ Swarm Nodes ได้ด้วยคำสั่ง

```bash
docker node ls
```

## Deploy Services

## Deploy Stacks

## Swarm Management

## Swarm vs Kubernetes
