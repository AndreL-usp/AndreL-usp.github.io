---
title: "Contribuição 1: Parte 1"
date: 2026-04-08
---

A partir do pad de sugestões, eu e minha dupla (Enzo Spinella) decidimos trabalhar na duplicação de código presente nas funções `jpeg_v5_0_1_process_interrupt` e `jpeg_v5_0_2_process_interrupt` dos arquivos `linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_1.c` e `linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_2.c`, respectivamente.

---

## Preparando o Ambiente de Desenvolvimento

O primeiro desafio que enfrentamos foi preparar o ambiente de desenvolvimento. Em um primeiro momento, minha dupla tentou, sem sucesso, criar um fork da tree da DRM AMDGPU presente em [https://gitlab.freedesktop.org/agd5f/linux.git](https://gitlab.freedesktop.org/agd5f/linux.git). 

Após isso, eu tentei apenas clonar a tree usando:

```bash
git clone https://gitlab.freedesktop.org/agd5f/linux.git --branch amd-staging-drm-next
```

Mas o download sempre falhava com código `503 (Serviço Indisponível)`. Na minha cabeça, eu estava clonando usando ssh, porque já tive problemas com https no github antes, mas, só depois de uma sugestão do monitor, percebi que estava usando https e ao tentar usar ssh, o download funcionou normalmente. 

### Limites no GitHub

Depois de conseguir clonar a tree, o próximo passo era colocar ela em um repositório nosso para poder trabalhar. Mas ao tentar dar push da tree no repositório do github, eu recebia o erro:

> `remote: fatal: pack exceeds maximum allowed size (2.00 GiB)`

Isso aconteceu porque a tree da AMDGPU é muito grande, e o github tem um limite de 2GB. Para resolver isso, eu tentei clonar a tree usando a opção `--depth 1` para pegar apenas o último commit, mas ao tentar dar push, eu recebia o erro:

> `remote: fatal: did not receive expected object ...`  
> `error: remote unpack failed: index-pack failed`

Isso aconteceu porque, ao usar `--depth 1`, eu não tinha o histórico completo da tree, e o github precisava de objetos que não estavam presentes no meu clone raso.

### A Solução: Migrando para o GitLab

A solução foi abandonar a ideia de usar o github e usar o gitlab próprio onde a tree da AMDGPU está hospedada e fazer um fork lá. 

Nesse momento, outro problema surgiu: o gitlab da AMDGPU exige autorização para criar forks e, curiosamente, para pedir autorização, é necessário criar uma issue. De qualquer forma, depois de criar a issue, a autorização foi concedida e eu consegui criar o fork.

Finalmente, temos um ambiente e repositório para trabalhar. 