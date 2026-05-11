---
title: Dirty Frag CVE-2026-31431.
authors: 
    - porucz 
date:
    created: 2026-05-06
tags:
    - linux
    - CVE-2026-31431
    - authencesn
---

## ¿Que es?

Dirty Frag es una vulnerabilidad de seguridad crítica descubierta en el kernel de Linux. Su nombre hace referencia a la forma en que manipula la fragmentación de paquetes en la memoria del sistema para causar un desbordamiento.

## ¿Cómo funciona?

El fallo reside en la gestión de los fragmentos de paquetes de red IPv6. Cuando un sistema recibe datos demasiado grandes para ser enviados de una sola vez, estos se dividen en trozos (fragmentos).
    
- El Problema: El kernel no valida correctamente los límites de memoria al reensamblar estos fragmentos.
- El Ataque: Un atacante puede enviar paquetes malformados que "engañan" al sistema.
- La Consecuencia: Se produce un desbordamiento de búfer (Buffer Overflow) en el heap del kernel.

## ¿Por qué es peligrosa?

Lo que hace que Dirty Frag sea especialmente preocupante es su alcance:

- Ejecución de Código Remoto (RCE): Un atacante podría, en teoría, ejecutar comandos con privilegios de administrador (root) de forma remota.
- Denegación de Servicio (DoS): Puede provocar el colapso total del sistema (kernel panic), dejando servidores fuera de línea.
- Sin autenticación: No se necesita una contraseña ni acceso previo al sistema; basta con poder enviar paquetes de red al objetivo.

## Mitigación

La solución es esperar el parche para el kernel, pero se puede mitigar inmediatamente con los siguientes comandos:

```bash
sh -c "printf 'install esp4 /bin/false\ninstall esp6 /bin/false\ninstall rxrpc /bin/false\n' > /etc/modprobe.d/dirtyfrag.conf; rmmod esp4 esp6 rxrpc 2>/dev/null; true"
```

Limpiar la paginación de la cache

```bash
echo 3 > /proc/sys/vm/drop_caches
```

```bash
rm /etc/modprobe.d/dirtyfrag.conf
```
