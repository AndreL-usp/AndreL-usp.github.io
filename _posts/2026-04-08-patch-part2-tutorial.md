---
title: "Contribuição 1: Parte 2"
date: 2026-04-08
---

## Analisando o Problema

Preparado o ambiente, chegou a hora de codar finalmente.

A primeira coisa que fizemos foi ler o código das funções `jpeg_v5_0_1_process_interrupt` e `jpeg_v5_0_2_process_interrupt` (que são iguais) para entender o que elas fazem e como elas estão estruturadas. A função é reproduzida abaixo:

```c
static int jpeg_v5_0_1_process_interrupt(struct amdgpu_device *adev,
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

Mas, as coisas estavam um pouco confusas. Então, decidimos entender o papel dos arquivos `jpeg_v5_0_1.c` e `jpeg_v5_0_2.c` na pasta `linux/drivers/gpu/drm/amd/amdgpu/` para entender melhor o contexto. A partir disso, conseguimos entender que esses arquivos são responsáveis por lidar com as interrupções do hardware de decodificação de JPEG da AMD, e que as funções `jpeg_v5_0_1_process_interrupt` e `jpeg_v5_0_2_process_interrupt` são chamadas quando uma interrupção é gerada por esse hardware. 

---

## Solução Adotada (Refatoração)

Entendido isso, partimos para pensar em como fazer a refatoração para eliminar a duplicação de código. A ideia que tivemos foi: Como as funções `jpeg_v5_0_1_process_interrupt` e `jpeg_v5_0_2_process_interrupt` são iguais, então, podemos: 

1. Criar um arquivo `linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_interrupt.c` que vai conter essa função;
2. Criar um header file `linux/drivers/gpu/drm/amd/amdgpu/jpeg_v5_0_interrupt.h` que declara essa função;
3. A partir daí, podemos colocar nos arquivos originais:
   ```c
   #include "jpeg_v5_0_interrupt.h"
   ```
   Isso vai permitir apenas chamar a função nos arquivos originais, ao invés de declará-la de forma duplicada. 
4. Ao final, precisaríamos incluir o novo arquivo no `Makefile` para garantir que ele seja compilado junto com os outros arquivos da pasta.

---

## Testando e Validando  

Implementamos isso e, para testar, encontramos um novo problema: 

A tree da AMDGPU não possui um `.config` pronto, então, não era possível compilar o kernel para testar a mudança. Conversando com o monitor, chegamos à conclusão de que precisaríamos criar um `.config` do zero, o que é algo bem complexo. Então, decidimos apenas confiar no pipeline de CI que foi implementado para a disciplina.

Após mandar o patch para esse pipeline, ele apontou que estava tudo certo:

![Captura de tela do pipeline](/assets/Captura%20de%20tela%20de%202026-04-20%2023-32-13.png)

Por fim, enviamos o patch para os mantenedores da AMDGPU e estamos aguardando o feedback deles.