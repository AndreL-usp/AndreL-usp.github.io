---
title: "Contribuição 1: Apresentação"
date: 2026-06-17
---

## O problema:

A task que pegamos foi deduplicar as funções 

```c
int jpeg_v5_0_1_process_interrupt(struct amdgpu_device *adev,
                                  struct amdgpu_irq_src *source,
                                  struct amdgpu_iv_entry *entry);
```

e

```c
int jpeg_v5_0_2_process_interrupt(struct amdgpu_device *adev,
                                  struct amdgpu_irq_src *source,
                                  struct amdgpu_iv_entry *entry);
```

presente nos arquivos 

```text
/linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_1.c
/linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_2.c
```

## Entendo o contexto desses arquivos:

Um driver é basicamente um programa que permite a comunicação entre o kernel do Linux e um hardware.

Os programas:

```text
/linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_1.c
/linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_2.c
```

fazem parte do driver da AMD (`amdgpu`) que estabelece a comunicação entre o kernel do Linux e uma placa de vídeo da AMD.

Mais especificamente, eles estabelecem a conexão entre o kernel do Linux e um bloco de hardware JPEG.

Acontece que uma placa de vídeo é dividida em blocos de hardware, cada bloco responsável por uma tarefa. Um bloco de hardware JPEG é responsável por codificar e decodificar imagens no formato JPEG.

Então, esses arquivos estão ligados com a codificação e decodificação de imagens JPEG.

Além disso, temos dois arquivos `jpeg_v5_0_1.c` e `jpeg_v5_0_2.c`, porque cada um lida com uma versão diferente do bloco de hardware JPEG.

---

## Entendido o papel dos arquivos, podemos enteder o papel das Funções Duplicadas

Nesses arquivos temos as funções duplicadas:

```c
int jpeg_v5_0_1_process_interrupt(struct amdgpu_device *adev,
                                  struct amdgpu_irq_src *source,
                                  struct amdgpu_iv_entry *entry);
```

e

```c
int jpeg_v5_0_2_process_interrupt(struct amdgpu_device *adev,
                                  struct amdgpu_irq_src *source,
                                  struct amdgpu_iv_entry *entry);
```

---


O papel delas é o seguinte:

Uma aplicação de usuário (como um navegador) precisa abrir um JPEG. Para isso, precisa decodificá-lo esse arquivo e para decodificar usa a placa de vídeo.

Para fazer essa comunicação com a placa, o driver `amdgpu` pega os dados e manda para a placa de vídeo, mas a CPU não fica bloqueada; ela continua executando normalmente.

Então, como ela sabe que a decodificação ficou pronta?

Quando a placa terminar de decodificar, ela manda um sinal para o kernel e o kernel chama a função `jpeg_v*_process_interrupt`, que por sua vez chama a função `amdgpu_fence_process()`.

```c

int jpeg_v5_0_1_process_interrupt(struct amdgpu_device *adev,
					 struct amdgpu_irq_src *source,
					 struct amdgpu_iv_entry *entry)
{
	u32 i, inst;

	i = node_id_to_phys_map[entry->node_id];
	DRM_DEV_DEBUG(adev->dev, "IH: JPEG TRAP\n");

	for (inst = 0; inst < adev->jpeg.num_jpeg_inst; ++inst)
		if (adev->jpeg.inst[inst].aid_id == i)
			break;

	if (inst >= adev->jpeg.num_jpeg_inst) {
		dev_WARN_ONCE(adev->dev, 1,
			      "Interrupt received for unknown JPEG instance %d",
			      entry->node_id);
		return 0;
	}

	switch (entry->src_id) {
	case VCN_5_0__SRCID__JPEG_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[0]);
		break;
	case VCN_5_0__SRCID__JPEG1_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[1]);
		break;
	case VCN_5_0__SRCID__JPEG2_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[2]);
		break;
	case VCN_5_0__SRCID__JPEG3_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[3]);
		break;
	case VCN_5_0__SRCID__JPEG4_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[4]);
		break;
	case VCN_5_0__SRCID__JPEG5_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[5]);
		break;
	case VCN_5_0__SRCID__JPEG6_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[6]);
		break;
	case VCN_5_0__SRCID__JPEG7_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[7]);
		break;
	case VCN_5_0__SRCID__JPEG8_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[8]);
		break;
	case VCN_5_0__SRCID__JPEG9_DECODE:
		amdgpu_fence_process(&adev->jpeg.inst[inst].ring_dec[9]);
		break;
	default:
		DRM_DEV_ERROR(adev->dev, "Unhandled interrupt: %d %d\n",
			      entry->src_id, entry->src_data[0]);
		break;
	}

	return 0;
}

```

A função `amdgpu_fence_process()` permite que a aplicação de usuário entenda que a placa terminou de decodificar e que a aplicação pode pegar os dados decodificados em algum lugar especificado.


## Entendido o contexto, podemos voltar ao problema

As funções:

```c
int jpeg_v5_0_1_process_interrupt(struct amdgpu_device *adev,
                                  struct amdgpu_irq_src *source,
                                  struct amdgpu_iv_entry *entry);
```

e

```c
int jpeg_v5_0_2_process_interrupt(struct amdgpu_device *adev,
                                  struct amdgpu_irq_src *source,
                                  struct amdgpu_iv_entry *entry);
```

São idênticas!! 

Solução proposta inicialmente (V1):

1. Criar um arquivo `linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_interrupt.c` que vai conter essa função;
2. Criar um header file `linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_interrupt.h` que declara essa função;
3. A partir daí, podemos colocar nos arquivos originais:
   ```c
   #include "jpeg_v5_0_interrupt.h"
   ```
   Isso vai permitir apenas chamar a função nos arquivos originais, ao invés de declará-la de forma duplicada. 
4. Ao final, precisaríamos incluir o novo arquivo no `Makefile` para garantir que ele seja compilado junto com os outros arquivos da pasta.



## Depois de três semanas sem feedback fizemos um resend e finalmente recebemos:

> "This is not common to jpeg v5.  jpeg_v5_0_0.c has a different interrupt handler, although it could probably be shared with that as well.  I think the additional fields in the IH should be 0s so it should work correctly, but we'd need to verify.  If you do want to share this, I would just make the implementation in jpeg_v5_0_1.c non-static and then just use it directly in jpeg_v5_0_2.c."

Pelo que entendi, o Alex argumenta que a função `jpeg_v5_0_1_process_interrupt` não é comum a todas as versões do driver JPEG v5, já que `jpeg_v5_0_0.c` (versão 5.0.0) tem um manipulador de interrupção diferente, apesar da `jpeg_v5_0_2.c` ter um igual. Ele também argumenta que talvez até fosse possível compartilhar a função com a versão 5.0.0, fazendo com que os campos adicionais no IH (Interrupt Handler) fossem `0`s, mas isso precisaria ser verificado. 

Então, ele sugere descartar a ideia de criar um `jpeg_v5_0_interrupt.c/h` do qual as versões 5.0.1 e 5.0.2 iriam pegar a função e simplesmente tornar a implementação da função `jpeg_v5_0_1_process_interrupt` não estática, para que ela possa ser usada diretamente em `jpeg_v5_0_2.c` através de uma declaração no header `jpeg_v5_0_1.h` e um `#include` desse header em `jpeg_v5_0_2.c`.



## Implementando o Patch v2

Então, nosso patch final ficou assim:

**Em** `jpeg_v5_0_1.c`, oque era estático:

```c
static int jpeg_v5_0_1_process_interrupt(struct amdgpu_device *adev,
				      struct amdgpu_irq_src *source,
				      struct amdgpu_iv_entry *entry)
```

Mudamos para não estático:

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

(Que era onde a função jpeg_v5_0_2_process_interrupt era usada)

**Por:**
```c
static const struct amdgpu_irq_src_funcs jpeg_v5_0_2_irq_funcs = {
    .set = jpeg_v5_0_2_set_interrupt_state,
    .process = jpeg_v5_0_1_process_interrupt
};
```

Além disso, retiramos o código referente a `jpeg_v5_0_2_process_interrupt`, eliminando, assim, a duplicação.

Feitas as mudanças, mandamos nosso patch v2 para o pipeline da disciplina. Ele foi aceito e então enviamos para os mantenedores da AMDGPU.





















