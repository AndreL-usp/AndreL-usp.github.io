---
title: "Quinto e Sexto tutoriais"
date: 2026-03-25
---

## Objetivos do Tutorial 5

Nesse tutorial, o foco foi entender como funciona o envio de patches no desenvolvimento do kernel, que é feito por email em vez de interfaces bonitas e intuitivas como pull requests do github.

A ideia principal é configurar e usar o git send-email para enviar commits / patches via email.

O tutorial começa pela instalação das dependências (como o pacote git-email no Ubuntu) e passa pela configuração básica do Git, incluindo nome e email do usuário.

Uma parte importante é a configuração do servidor SMTP, onde você define coisas como:

endereço do servidor (ex: smtp.gmail.com)
porta (geralmente 587)
tipo de criptografia (tls)
usuário (seu email)

No caso do Gmail, não dá mais para usar a senha normal. É necessário ativar autenticação em dois fatores (2FA) e gerar uma App Password, que é uma senha específica de 16 caracteres.

Uma coisa importante é que quando eu gerei a App Password, não ficou claro que ela seria mostrada apenas uma vez. Como eu tenho costume de anotar senhas, depois consegui usar ela sem problemas. Mas, para quem não anotou, perdeu hehe.

Depois disso, o tutorial mostra como enviar patches na prática. Para algo simples, dá para usar:

git send-email -1 --to="bla@bla.com" --cc="bla2@bla.com"

Para casos mais completos, entram opções como:

--annotate: abre o email no editor para revisão antes de enviar
--cover-letter: cria um email introdutório explicando o conjunto de mudanças


## Problemas Encontrados e Soluções no Tutorial 5:

Não tive nenhum grande problema nesse :)

## Objetivos do Tutorial 6

Esse tutorial tem como objetivo resolver o problema de usar git send-email com contas da USP. Esse problema acontece porquecontas da USP não permitem ativar 2FA da forma padrão, então não dá para gerar App Passwords como em contas normais do Gmail que seriam usadas para autenticar o envio de emails.

A solução é usar um email proxy com OAuth, rodando via Docker. Nesse caso, subimos um container com um serviço que autentica usando OAuth com tokens disponibilizados pelo monitor, e fazemos o git send-email usar esse proxy para enviar os emails. Mas, aparentemente, para fazer isso, é necessário dar permissões para o proxy acessar a conta da USP.

Acabei optando por não usar meu email pessoal da USP para fazer as submissiões e usar as coisas do tutorial 5 ao invés disso, porque não gostei do fato de ter que dar autorizações sobre minha conta da USP para poder usar o proxy. Sinto muito.


## Problemas Encontrados e Soluções no Tutorial 6:

Não tive nenhum grande problema nesse além do dilema de usar ou não minha conta da USP para isso :)