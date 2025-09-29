---
title: Notes
description: Notas gerais sobre configurações, problemas complexos, configurações esporádicas entre outras coisas importantes para lembrar em algum momento :)
date: '2022-04-22'
aliases:
  - referencia
  - reference
  - ref
license: CC BY-NC-ND
lastmod: '2022-04-22'
menu:
    main: 
        weight: -60
        params:
            icon: book
image: photo-1533279443086-d1c19a186416.jpg
---

### Sonarqube + WSL2

Para executar o container do sonarqube localmente com o wsl2 é necessário aumentar a memória da jvm no wsl para que seja possível executar o elasticsearch.

```bash
# Arquivo \.wslconfig
[wsl2]
kernelCommandLine = "sysctl.vm.max_map_count=262144"
```

```yaml
# sonarqube-service.yml

version: "3"
services:
  sonarqube:
    image: sonarqube:community
    depends_on:
      - db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ulimits:
      nproc: 131072
      nofile:
        soft: 8192
        hard: 131072
    ports:
      - "9100:9000"
  db:
    image: postgres:12
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:
```



### WSL2 - Inicializar serviços no startup do wsl

```bash
# Enable systemd

echo -e "[boot]\nsystemd=true" | sudo tee /etc/wsl.conf
```

```bash
# vim /etc/wsl.conf

[boot]
command="service docker start && service ssh start"
```

### Forçar sincronização do Azure AD no Windows Server

```powershell
# Powershell as admin

Import-Module ADSync
Start-ADSyncSyncCycle -PolicyType Delta
# Full sync
# Start-ADSyncSyncCycle -PolicyType Initial
```

### Tasmota Zigbee Hub
- 12.5.0(tasmota)
- ESP 5741884 (ESP32-D0WD-V3 rev.3)

console
```
// Habilita Ntp Server
time 0

// Habilita MQTT
SetOption3 1

// Configura topic mqtt único por dispositivo
SetOption89 1
```