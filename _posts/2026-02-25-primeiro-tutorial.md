---
title: "Primeiro tutorial"
date: 2026-02-25
---

Comecei o primeiro tutorial da disciplina animado para aprender mais sobre o funcionamento do kernel e sobre o uso de VM's para contribuir com o desenvolvimento do kernel.

Esse primeiro tutorial teve como objetivo criar um ambiente seguro de testes para o desenvolvimento do kernel usando máquinas virtuais. Para isso, ele mostra como configurar uma VM com QEMU e libvirt, permitindo compilar e testar kernels modificados sem afetar o sistema principal e também mostra como acessar essa máquina virtual com SSH.

Alguns problemas que encontrei ao seguir o tutorial foram: o link para a imagem do Debian estava quebrado e tive que encontrar uma alternativa, os comandos do libguestfs não estavam com sudo no tutorial e eu precisei usar, tive algumas dificuldades com o dpkg-reconfigure openssh-server e a imagem de Debian que baixei não tinha rsync fazendo com que eu tivesse que rodar "apt install rsync" na VM. De resto, tudo ocorreu bem e pude aprender bastante.
