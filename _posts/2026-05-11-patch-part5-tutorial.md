---
title: "Contribuição 1: Parte 5 - Patch v2"
date: 2026-06-10
---

## Feedback dos Mantenedores

Retomando, o feedback do Alex foi o seguinte:

> "This is not common to jpeg v5.  jpeg_v5_0_0.c has a different interrupt handler, although it could probably be shared with that as well.  I think the additional fields in the IH should be 0s so it should work correctly, but we'd need to verify.  If you do want to share this, I would just make the implementation in jpeg_v5_0_1.c non-static and then just use it directly in jpeg_v5_0_2.c."

Pelo que entendi, o Alex argumenta que a função `jpeg_v5_0_1_process_interrupt` não é comum a todas as versões do driver JPEG v5, já que `jpeg_v5_0_0.c` (versão 5.0.0?) tem um manipulador de interrupção diferente, apesar da `jpeg_v5_0_2.c` ter um igual. Ele também argumenta que talvez até fosse possível compartilhar a função com a versão 5.0.0, fazendo com que os campos adicionais no IH (Interrupt Handler) fossem `0`s, mas isso precisaria ser verificado. 

Então, ele sugere descartar a ideia de criar um `jpeg_v5_0_interrupt.c/h` do qual as versões 5.0.1 e 5.0.2 iriam pegar a função e simplesmente tornar a implementação da função `jpeg_v5_0_1_process_interrupt` não estática, para que ela possa ser usada diretamente em `jpeg_v5_0_2.c` através de uma declaração no header `jpeg_v5_0_1.h` e um `#include` desse header em `jpeg_v5_0_2.c`.

Eu acho meio feio estar no `jpeg_v5_0_2.c` e chamar a função `jpeg_v5_0_1_process_interrupt`, mas foi o que ele sugeriu.

---

## Implementando o Patch v2

Então, nosso patch final ficou assim:

**Em** `jpeg_v5_0_1.c`, mudamos para não estático:
```c
int jpeg_v5_0_1_process_interrupt(struct amdgpu_device *adev,
                                  struct amdgpu_irq_src *source,
                                  struct amdgpu_iv_entry *entry)
```

**Em** `jpeg_v5_0_1.h`, adicionamos a declaração da função:
```c
int jpeg_v5_0_1_process_interrupt(struct amdgpu_device *adev,
                                  struct amdgpu_irq_src *source,
                                  struct amdgpu_iv_entry *entry);
```

**E em** `jpeg_v5_0_2.c`, adicionamos o `#include "jpeg_v5_0_1.h"` e substituímos a chamada da função `jpeg_v5_0_2_process_interrupt` em:

```c
static const struct amdgpu_irq_src_funcs jpeg_v5_0_2_irq_funcs = {
    .set = jpeg_v5_0_2_set_interrupt_state,
    .process = jpeg_v5_0_2_process_interrupt
};
```

**Por:**
```c
static const struct amdgpu_irq_src_funcs jpeg_v5_0_2_irq_funcs = {
    .set = jpeg_v5_0_2_set_interrupt_state,
    .process = jpeg_v5_0_1_process_interrupt
};
```

Feitas as mudanças, mandamos nosso patch v2 para o pipeline da disciplina. Ele foi aceito e então enviamos para os mantenedores da AMDGPU.



