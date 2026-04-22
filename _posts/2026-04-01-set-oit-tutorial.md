---
title: "Sétimo e Oitavo tutoriais"
date: 2026-04-01
---

## Objetivos do Tutorial 7

Nesse tutorial, o foco foi entender a estrutura interna de um driver do subsistema IIO do kernel, entender como drivers de sensores são organizados, a definição de channels, que representam medições como tensão e aceleração e como eles são configurados usando a struct iio_chan_spec.

Após isso, foi abordado funções de leitura e escrita (iio_dummy_read_raw() e iio_dummy_write_raw()) para manipular os dados dos sensores.

Achei um pouco mais teórico que os anteriores, mas sobrevivemos.


## Objetivos do Tutorial 8

A ideia desse tutorial foi colocar em prática o que aprendemos no tutorial anterior modificando o driver iio_dummy para adicionar novos canais referentes a uma bússola com 3 eixos (X, Y, Z).

A primeira parte é bem direta: habilitar o iio_dummy no .config, compilar só aquele módulo com
make M=drivers/iio/dummy, instalar com make modules_install e carregar com modprobe. Depois vemos o uso de configfs para criar dinamicamente um dispositivo dummy e, então, modificar o driver para adicionar os canais que queremos.

## Problemas Encontrados e Soluções no Tutorial 8:

Aparentemente, o tutorial usa uma versão mais antiga do iio_dummy e naas versões mais recentes a função iio_dummy_read_raw() foi mudada e parte do seu trabalho foi delegado para funções auxiliares como a __iio_dummy_read_raw(). A solução foi abrir o código do iio_simple_dummy.c e identificar onde a lógica realmente está sendo tratada.

Acabaram os tutoriais :0