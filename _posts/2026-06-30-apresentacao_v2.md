---
title: "Fase 2: Apresentação"
date: 2026-07-08
---

# Atualização sobre a contribuição para o kernel:

## Histórico:
Fluxo:

```text
V1
Sem resposta
V1 Resend
Feedback
V2 na mesma thread da V1 Resend
Sem resposta
V2 Resend na mesma thread da V1 Resend
Sem resposta
```

Como ainda estamos sem resposta, foi sugerido pelos monitores de mandar a V2 numa thread diferente. Fizemos isso e estamos esperando se dessa vez teremos uma resposta.

# Fase 2:

## Busca por uma contribuição

A busca foi muito simples: tivemos em aula a apresentação sobre o projeto do ArKanjo e nela o Guilherme Ivo, um dos mantenedores, apresentou algumas issues que inclusive já estavam descritas e categorizadas no repositório do github do projeto e, dentre elas, foi apresenta a issue #31 que pedia a adição da opção / flag --all no comando preprocessor clean.

![cc](/assets/g6.png)

A motivação é simples: O Arkanjo usa uma etapa de pre-processamento para acelerar consultas posteriores de duplicacao de codigo. Essa etapa cria um cache em disco que podemos querer limpar completamente por motivos de espaço etc.

Por ser uma issue interessante, bem documentada, marcada como good first issue e com o mantenedor ao nosso alcance, decidimos atacá-la.

## V1 (nossa primeira tentativa)

Como o comando preprocessor clean já havia sido feito e já havia a opção --name, conseguimos reutilizar parte da lógica, mas houve uma grande dificuldade em entender como registrar a opção --all porque no código onde era implementado o comando clean (preprocessor_clean.cpp), não havia um registro explícito da opção --name.

Dessa forma, foi tranquilo implementar a feature, mas estava sendo difícil registrar ela.

Então, vimos que havia a opção verbose em algum lugar, a partir disso, buscamos pela palavra verbose no projeto em encontramos o vetor de structs:

```c++
  public:
    static constexpr CliOption options_[] = {
      {"verbose", 0, NoArgument, "Enable verbose output"},
      {"path", 0, PositionalArgument, "Project path to preprocess."},
      {"minimum-lines", 0, RequiredArgument, "Minimum clone size in original lines."},
      OPTION_END
    };
```

No arquivo preprocessor_build.cpp

Com isso, conseguimos descobrir como registrar a opção --all e foi possível terminar nossa V1.

### A solução que pensamos:

![Captura de tela do pipeline](/assets/b.png)

![Captura de tela do pipeline](/assets/c.png)


Basicamente, usamos o map options.args para verificar se foi passado a opção --all ou não e se sim, usamos a variável base_path que contém o caminho para o diretório com o cache e simplesmente iteramos pelas entradas desse diretório apagando tudo pelo caminho.

### Feedback da V1:

Implementado, fizemos um PR no repositório do ArKanjo e recebemos rapidamente o seguinte feedback:

![Captura de tela do pipeline](/assets/d.png)

![Captura de tela do pipeline](/assets/e.png)

![Captura de tela do pipeline](/assets/f.png)


que pode ser resumido em:

- Signed-off-by só é necessário quando Developer Certificate of Origin (DCO) é requerido no projeto. Como o Arkanjo parece não requerer DCO, não precisávamos usar Signed-off-by no nosso commit.

- Ao invés de validar o uso da flag --all dentro do método PreprocessorClean::run(), deveríamos ter usado um atributo (clean_all) na classe PreprocessorClean que indica se a flag foi utlizada ou não e dentro do método PreprocessorClean::validate() validar esse uso usando 

```c++
clean_all = options.args.count("all") > 0
```

- Ao invés de iterar por cada entrada no diretório, deveríamos remover o diretório e depois recriá-lo, por questões de otimização. Além de reutilizar a lógica de remoção que já estava sendo feita.

- Também foi pedido para mudarmos a formatação de:

```c++
static constexpr CliOption options_[] = { {"all", 0,NoArgument, "Remove all cached preprocess profiles."},OPTION_END};
```

Para:

```c++
static constexpr CliOption options_[] = {
    {"all", 0, NoArgument, "Remove all cached preprocess profiles."},
    OPTION_END
};
```

O que ajuda a adicionar novas opções sem ter que modificar a mesma linha, o que preserva o git blame.

## V2:

Aplicamos as sugestões, e geramos o seguinte commit:

![a](/assets/g1.png)

![a](/assets/g2.png)

![a](/assets/g3.png)

Feito o commit, atualizamos nosso Pull Request e esperamos por feedback. 


### Feedback da V2:

```c++
Thanks! I applied the suggested approach and switched back to iterating over
the directory contents instead of removing the cache directory itself.
This avoids removing the symlink when the cache directory is a symbolic link.
I also resolve the cache path with fs::weakly_canonical() before cleaning.
Guilherme Ivo
```

Basicamente, a ideia de apagar o diretório de cache e depois recriá-lo poderia dar problema, caso o caminho usado pelo método clean fosse um link simbólico pq nesse caso vc estaria apagando o link e criando um diretório no lugar, fazendo o programa perder a referência para o diretório original.

Além disso, o fs::weakly_canonical() torna mais consistente o caminho, o que pode ser uma boa antes de modificar ele.

Após esse comentário, o Guilherme fechou a issue e adicionou um commit com nossas modificações :)

# Nova contribuição para o ArKanjo ?

![a](/assets/g5.png)