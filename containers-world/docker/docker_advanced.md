
### ✅ Modern Compose

```bash
docker-compose up
```

- Defined using `docker-compose.yml`
- Supports service orchestration, dependency management, and networking.

---

## 🔌 Networking in Docker Compose

### Networks Property

```yaml
networks:
  frontend:
  backend:
```

### Custom Network with Services

```yaml
services:
  app:
    image: myapp
    networks:
      - frontend

  redis:
    image: redis
    networks:
      - backend

networks:
  frontend:
  backend:
```

- Default network is created by Compose.
- `networks` block allows segregation of service traffic.

---

## 🔗 Linking vs Compose Networking

| Feature          | Legacy `--link`         | Compose v2+ Networking   |
|------------------|--------------------------|---------------------------|
| Scope            | Local host only          | Cross-host (Swarm supported) |
| DNS              | Static name mapping      | Dynamic DNS discovery     |
| Preferred        | ❌ Deprecated             | ✅ Recommended             |

---

## ⚙️ `depends_on` in Docker Compose

```yaml
services:
  web:
    depends_on:
      - db
```

- Ensures `db` starts **before** `web`.
- Does **not** wait for service to be ready, just launched.

> Use wait-for-it.sh or healthchecks for readiness.

---

## 🔢 Compose File Version 3 (v3)

- Introduced for compatibility with Docker Swarm mode.
- Adds `deploy`, `replicas`, and `placement` constraints.
- Drops `depends_on` behavior from v2 (must use healthchecks).

---

## ⚙️ Docker Swarm (Native Orchestration)

```bash
docker swarm init
docker swarm join --token <TOKEN> <MANAGER-IP>:<PORT>
```

### Features

- Built-in orchestration engine.
- Supports service discovery, scaling, and self-healing.

---

## 🛡️ Raft Consensus & Quorum

- Swarm uses **Raft** protocol for leader election.
- Minimum **quorum** of manager nodes required for decisions.

| Managers | Quorum |
|----------|--------|
| 3        | 2      |
| 5        | 3      |
| 1        | 1 (edge case) |

If quorum breaks (e.g., 2/3 managers fail), force recovery:

```bash
docker swarm init --force-new-cluster --advertise-addr <ip>
```

---

## 🧮 Node Management in Swarm

```bash
docker node ls                      # List all nodes
docker node promote <node-name>     # Promote to manager
docker swarm join-token manager     # Get token to join as manager
docker swarm join-token worker      # Get token to join as worker
docker swarm leave                  # Leave cluster
```

---

## 📘 `.env` File Best Practice

When adding new nodes (worker or manager), ensure:

- Update `.env` or `.env.production` with appropriate host IPs
- Update any DNS/Load Balancer entries if required

---

## 📚 References

- [Docker Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Docker Networking](https://docs.docker.com/network/)
- [Docker Swarm Mode](https://docs.docker.com/engine/swarm/)
- [Understanding Raft](https://raft.github.io/)

