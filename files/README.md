# Tesla P40 monitoring stack

Prometheus + Grafana + node_exporter + nvidia_gpu_exporter, wired together and
pre-provisioned with a P40 dashboard. Zero clicks after `up`.

## Prereqs (already done in your build)
- Docker + nvidia-container-toolkit installed
- `nvidia-ctk runtime configure --runtime=docker` has run (registers the `nvidia` runtime)
- `nvidia-smi` shows both P40s

## Deploy
    cd monitoring
    # edit docker-compose.yml: change GF_SECURITY_ADMIN_PASSWORD first
    docker compose up -d
    docker compose ps          # all four containers should be "running"

## Verify the plumbing
    # GPU exporter is producing metrics:
    docker exec nvidia_gpu_exporter wget -qO- http://localhost:9835/metrics | grep nvidia_smi_temperature_gpu
    # Prometheus sees its targets as UP:
    #   open http://127.0.0.1:9090/targets  (via tunnel, see below)

## Reaching Grafana (it is bound to 127.0.0.1 on purpose)
Docker publishes ports by writing iptables rules that BYPASS ufw, so anything
mapped to 0.0.0.0 would be exposed regardless of your firewall. To avoid that,
Grafana and Prometheus are bound to localhost only. Reach them by either:

  A) SSH tunnel from your laptop:
        ssh -L 3000:localhost:3000 -L 9090:localhost:9090 wofl@<server>
     then browse http://localhost:3000

  B) Tailscale: change the port line to your tailscale IP, e.g.
        ports: ["100.x.y.z:3000:3000"]
     then browse http://100.x.y.z:3000 from any tailnet device.

Login: admin / (the password you set). Dashboard: "Tesla P40 — GPU & Host Monitoring"
auto-loads under Dashboards.

## Stop / update
    docker compose down            # stop (keeps data volumes)
    docker compose pull && docker compose up -d   # update images
