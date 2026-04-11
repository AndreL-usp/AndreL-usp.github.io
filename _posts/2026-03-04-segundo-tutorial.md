---
title: "Segundo tutorial"
date: 2026-03-04
---

## Objetivos do Tutorial

Nesse tutorial 2, o objetivo principal foi aprender a compilar e testar um kernel Linux customizado para arquitetura ARM64 usando máquinas virtuais. A ideia é pegar o código-fonte do kernel, configurar o que será incluído, compilar tudo com cross-compilation e rodar esse kernel dentro de uma VM.

Além disso, o tutorial mostra como usar o **kw (kworkflow)** para automatizar várias partes do processo, como configuração, build e deploy dos módulos, deixando o fluxo mais prático para desenvolvimento.

No fim, a proposta é criar um ambiente de testes onde você consegue modificar o kernel, recompilar e validar suas mudanças rapidamente dentro da VM.

---

## Problemas Encontrados e Soluções

Durante o processo, encontrei os seguintes problemas:

### 1)

Tive um problema ao executar:
```bash
kw deploy --modules
```

Que gerou a seguinte mensagem de erro:
> `E: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 395 (unattended-upgr)`

Basicamente a VM resolveu atualizar coisas no momento que eu ia instalar dependências e aí deu erro.

### 2)

Depois de resolver eu tentei novamente `kw deploy --modules` e deu o erro:
> `strip: Não foi possível reconhecer o formatado de entrada do arquivo ...`

Para resolver eu tentei colocar:
`export STRIP=aarch64-linux-gnu-strip` no `./active.sh` que vinha sendo usado mas não funcionou, então pesquisando eu cheguei a esse comando:

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_STRIP=modules_install INSTALL_MOD_PATH=tmp/arm64_modules
```

Após isso as coisas funcionaram mas aí eu cheguei em outro problema.

### 3)

Ao executar `sudo virsh console arm64` aparece `"Connected to domain 'arm64' Escape character is ^] (Ctrl + ])"` e não passa disso.

Pesquisando eu cheguei em uma solução e fiz um mini tutorial para resolver esse problema, que é o seguinte:

No `create_vm_virsh()`, colocar:
```bash
kernel_args="console=ttyAMA0 loglevel=8 root=/dev/vda2 rootwait"
```

No `launch_vm_qemu()`, colocar:
```bash
-append "console=ttyAMA0 loglevel=8 root=/dev/vda2 rootwait"
```

Depois execute:
```bash
kw build --menu
```

Depois navegue em:
1. `Device Drivers`
2. `Character devices`
3. `Serial drivers`

Habilite:
```text
[*] ARM AMBA PL011 serial port support
[*]   Console on AMBA PL011 serial port
```

Salve e saia.

Depois execute o comando:
```bash
kw build
```

Depois execute recursivamente os comandos para recriar a VM:
```bash
sudo virsh destroy arm64
sudo virsh undefine arm64
create_vm_virsh
```

E agora `sudo virsh console arm64` deveria estar funcionando!