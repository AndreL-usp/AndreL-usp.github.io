---
title: "Conclusão: Contribuições para o Kernel e para o ArKanjo (fase 2) concluídas"
date: 2026-07-10
---


Ao final do semestre, nossas contribuições para o kernel e para o ArKanjo foram ambas aceitas:

# Aceite da contribuição para o Kernel:


```text
Applied.  Thanks!

Alex Deucher 
```

# Aceite da contribuição para o ArKanjo (fase 2):



```c++
Thanks! I applied the suggested approach and switched back to iterating over
the directory contents instead of removing the cache directory itself.
This avoids removing the symlink when the cache directory is a symbolic link.
I also resolve the cache path with fs::weakly_canonical() before cleaning.

Guilherme Ivo
```

Com isso, finalizamos as contribuições e posts da primeira e segunda fase da disciplina !!