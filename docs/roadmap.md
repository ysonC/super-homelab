## Todo List

- [ ] Move vaultwarden back to longhorn
- [ ] Implement SQL database for Prometheus and Grafana.

### CI/CD Automation

- [ ] Implement GitHub Actions to run linting, validation, and testing.
- [ ] Add automated tests for basic health checks.

### Prometheus and Grafana: Stuff to add

- [x] Machine uptime.
- [x] CPU usage for each service.
- [x] Memory usage for each service.
- [ ] Implement alert rules for each service.
- [ ] Logs integreation with Loki.

### Backup and Disaster Recovery

- [ ] Implement backup and disaster recovery for each service.

## Completed

- [x] Implement new storage class for nfs.
- [x] Implement SQL database for Immich.

### Ansible

- [x] Revisit Ansible playbooks and roles.

- [x] Implement SQL database for Vaultwarden.
- [x] Implement Renovate for automated dependency updates.

### n8n

- [x] Add check for breaking change from Renovate
- [x] Add pipline to check every PR from Renovate to check if update require additional change to yaml
