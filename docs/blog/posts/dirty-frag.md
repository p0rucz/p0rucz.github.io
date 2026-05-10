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

## Mitigación

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
