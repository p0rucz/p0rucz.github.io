---
title: Migración de adguard home en lxc
author: 
    - porucz 
date: 
    created: 2026-05-11
tags: 
    - linux
    - lxc
    - proxmox
---

## Respaldar configuración

Listar contenedores en LXC

```bash
pct exec ID -- tar czvf /root/AdGuardHome.tar.gz /opt/AdGuardHome
```

Obtener el archivo tar 

```bash
pct pull ID /root/AdGuardHome.tar.gz /root/AdGuardHome.tar.gz
```

## Detener servicio.

```bash
pct stop ID
```

## Migrar información al nuevo contenedor LXC

Subir el archivo al contenedor

```bash
pct push ID /root/AdGuardHome.tar.gz 
```

Detener servicio en el nuevo contenedor 

```bash
pct exec ID -- pkill AdGuardHome
``` 

Descomprimir archivo 

```bash 
pct exec ID -- tar xcvf AdGuardHome.tar.gz -C /opt
```

Reiniciar el contenedor 

```bash
pct reboot ID
```
