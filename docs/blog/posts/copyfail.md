---
title: Copyfail CVE-2026-31431.
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

Se ha dado a conocer una vulnerabilidad crítica bautizada como "Copy Fail" (registrada bajo el CVE-2026-31431) por parte de [xint](https://xint.io/blog/copy-fail-linux-distributions), que permite a cualquier usuario local obtener privilegios de root de forma determinista y silenciosa. A diferencia de otros fallos históricos, este no depende de condiciones de carrera (race conditions), lo que lo hace extremadamente fiable y peligroso.

<!-- more -->

Es un fallo de lógica en el subsistema criptográfico del kernel de Linux (específicamente en el módulo algif_aead). La vulnerabilidad permite que un atacante escriba 4 bytes de datos controlados directamente en la caché de páginas (page cache) de cualquier archivo que pueda leer, incluso si no tiene permisos de escritura sobre él.

Al corromper la memoria que el sistema usa para ejecutar binarios privilegiados (como /usr/bin/su o /usr/bin/sudo), un atacante puede inyectar código malicioso que se ejecuta con los máximos privilegios.

## Puntos Clave

- Alcance: Afecta a prácticamente todas las distribuciones principales (Ubuntu, RHEL, Debian, Amazon Linux, SUSE) con kernels lanzados desde 2017.
- Sin rastro en el disco: La corrupción ocurre solo en la memoria RAM. El archivo físico en el disco permanece intacto, lo que hace que muchas herramientas de integridad de archivos no detecten el ataque.
- Escape de Contenedores: Debido a que el kernel y la caché de páginas se comparten entre el host y los contenedores, un atacante dentro de un contenedor podría comprometer a toda la máquina anfitriona.

## ¿Cómo saber si eres vulnerable?

Puedes verificar la versión de tu kernel y si el módulo afectado está cargado con estos comandos:

Listar el kernel en uso

```Bash
uname -r
```

Kernels que han sido probados y son vulnerables: 6.12, 6.17, y 6.18

Verificar si el módulo algif_aead está cargado

```bash
lsmod | grep algif_aead
```

## Mitigación inmediata

Mientras llegan los parches oficiales de tu distribución, la recomendación principal es deshabilitar el módulo afectado

```bash 
rmmod algif_aead
```
Para evitar que se cargue al arranque, crea un archivo en `/etc/modprobe.d/block_algif.conf` con el contenido:

```ruby
blacklist algif_aead
```

También puede bloquearse este modulo mediante la configuración de grub. Añadiendo `modprobe.blacklist=af_alg`:

```ruby
GRUB_CMDLINE_LINUX_DEFAULT="modprobe.blacklist=af_alg"
```

Para que este cambio surta efecto es necesario volver a construir la configuración del grub.

```bash 
grub-mkconfig -o /boot/grub/grub.cfg
```

!!! warning
    Deshabilitar este módulo puede afectar a aplicaciones que usen aceleración criptográfica por hardware a través del kernel, aunque la mayoría de las apps modernas tienen métodos alternativos en espacio de usuario.
