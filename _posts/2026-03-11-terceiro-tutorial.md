---
title: "Terceiro tutorial"
date: 2026-03-11
---

Mais um tutorial, então mais um post :)
## Objetivos do Tutorial

Nesse tutorial, o objetivo foi introduzir o desenvolvimento no kernel Linux através da criação e uso de módulos, mostrando como adicionar novas funcionalidades sem precisar recompilar todo o kernel.

A ideia é aprender a criar um módulo simples, configurar o sistema de build do kernel usando `Kconfig` e `Makefile`, compilar e carregar esse módulo na VM, além de entender como ativar funcionalidades com o `menuconfig`.

Também são explorados conceitos importantes como dependência entre módulos e carregamento dinâmico usando `insmod` e `modprobe`.

---

## Problemas Encontrados e Soluções

No geral, deu tudo certo nesse tutorial, mas encontrei novamente o problema do tutorial 2 em que ao rodar:

```bash
sudo virsh console arm64
```

Fica travado na mensagem:
> `"Connected to domain 'arm64' Escape character is ^] (Ctrl + ])"`

Pesquisando, parece que ao rodar:

```bash
kw build --clean
```

As configurações que eu tinha feito no `kw build --menu` foram perdidas, o que gerou o travamento do console. Para resolver isso, bastou refazer o mini tutorial do tutorial 2.