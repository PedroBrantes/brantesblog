---
title: Resolvendo problema de deploy no hugo e netlify
date: 2024-03-01 20:39:46 -03:00
description: "Command failed with exit code 1: hugo (https://ntl.fyi/exit-code-1)"
author: Eu
tags:
  - blogdown
  - netlify
  - obsidian
  - markdown
  - rmarkdown
  - r
  - hugo
---
# Como Foi o Dia

Hoje passei lendo documentação sobre o [blogdown](https://github.com/rstudio/blogdown) para resolver um problema "bobo" de deploy, pois o `netlify` estava apresentando um erro de sintaxe no HTML que é renderizado pelo `hugo` (Gerador de sites estáticos). Ao fazer a depuração, imaginei que o problema estava relacionado à conversão de arquivos markdown `(.md | .markdown)` para HTML. Cheguei até a pensar que fosse o R Markdown que é renderizado pela função `blogdown::build_dir()`, que também chama `rmarkdown::render()` e faz a conversão para arquivos MD. Segue a tabela:

| Feature          | .Rmd | .Rmarkdown | .md/.markdown |
| :--------------- | :--- | :--------- | :------------ |
| Run R code       | yes  | yes        | no            |
| Bibliography     | yes  | yes        | no            |
| Cross references | yes  | yes        | no            |
| LaTeX math       | yes  | maybe      | maybe         |
| HTML widgets     | yes  | yes        | no            |
Fonte Tabela: [Seção 1.6](https://bookdown.org/yihui/blogdown/output-format.html#output-format)

O erro que estava sendo apresentado era o seguinte:

```
1:33:09 PM: $ hugo
1:33:09 PM: Start building sites …
1:33:09 PM: hugo v0.123.6-92684f9a26838a46d1a81e3c250fef5207bcb735+extended linux/amd64 BuildDate=2024-02-28T18:29:40Z VendorInfo=gohugoio
1:33:09 PM: Total in 41 ms
1:33:09 PM: Error: error building site: render: failed to render pages: failed to process "/post/primeiro/index.html": "/tmp/hugo-transform-error2030905505💯40": expected comma character or an array or object ending on line 100 and column 40
1:33:09 PM:    12:     {
1:33:09 PM:            ^
1:33:09 PM: ​
1:33:09 PM: "build.command" failed                                        
1:33:09 PM: ────────────────────────────────────────────────────────────────
1:33:09 PM: ​
1:33:09 PM:   Error message
1:33:09 PM:   Command failed with exit code 1: hugo (https://ntl.fyi/exit-code-1)
1:33:09 PM: ​
1:33:09 PM:   Error location
1:33:09 PM:   In build.command from netlify.toml:
1:33:09 PM:   hugo
1:33:09 PM: ​
1:33:09 PM:   Resolved config
1:33:09 PM:   build:
1:33:09 PM:     command: hugo
1:33:09 PM:     commandOrigin: config
1:33:09 PM:     environment:
1:33:09 PM:       - HUGO_VERSION
1:33:09 PM:       - HUGO_ENV
1:33:09 PM:     publish: /opt/build/repo/public
1:33:09 PM:     publishOrigin: config
1:33:09 PM: Build failed due to a user error: Build script returned non-zero exit code:2
1:33:09 PM: Failing build: Failed to build site
1:33:09 PM: Finished processing build request in 21.679s
1:33:09 PM: Failed during stage "building site": Build script returned non-zero exit code: 2´
```

Como podem perceber, na linha:

```
1:33:09 PM: Error: error building site: render: failed to render pages: failed to process "/post/primeiro/index.html": "/tmp/hugo-transform-error2030905505💯40": expected comma character or an array or object ending on line 100 and column 40
```

A causa do problema parece ser óbvia. Como mencionei anteriormente, um erro de sintaxe ao realizar a renderização/conversão para HTML. Porém, este não era o verdadeiro problema. Realizei diversos testes (excluí certas pastas, mudei alguns parâmetros dentro do arquivo de texto, etc.), mas sem sucesso. Então, fui em busca de documentações e encontrei alguns artigos muito úteis. Vou deixar links no final.
## Resolução do Problema

Bom, indo direto ao ponto, o problema estava relacionado a uma variável do `hugo` no arquivo `config.yaml` (pode ser `config.toml`). A variável era `baseurl` e o valor padrão era `/`:

```yaml
baseurl: /
title: Brantes
```

Alterando para `baseurl: https://brantes.netlify.app` e realizando push para o repositório, o deploy foi realizado com sucesso!

Esta simples mudança também foi útil para definir onde os arquivos de mídia seriam armazenados no meu **workflow** que montei pelo ``Obsidian`` para criar os posts. Segue o exemplo:

![exemplo obsidian](../../../img/Pasted%20image%2020240301214848.png)

Todos os arquivos que estão na pasta `/static` são movidos para a raiz `/public` ao fazer o deploy. Então, eu defini para que o ``Obsidian`` já armazenasse qualquer arquivo na pasta `/public/img`:

![exemplo obsidian 2](../../../img/Pasted%20image%2020240301220312.png)

Assim, eu apenas preciso referenciar por níveis `../`. Cada `../` representa um subdiretório ou pasta anterior/superior. Como a minha pasta de `/content/post` se transforma em `/Post` após o deploy, os links podem ser referenciados assim:

```
![alt](../../../img/Pasted%20image%2020240301214848.png)
```

Também é preciso desativar os Wikilinks do Obsidian, pois não são suportados pelo Hugo:
![desativar wikilinks obsidian](../../../img/Pasted%20image%2020240301220810.png)

Enfim, espero que este artigo/blog tenha sido útil em algum nível. Abaixo, vou deixar links para discussões e documentações que me ajudaram.

# Links Relacionados

- [github.com](https://github.com/rstudio/blogdown/issues/45)
- [bookdown.org](https://bookdown.org)
    - [static-files](https://bookdown.org/yihui/blogdown/static-files.html)
    - [html](https://bookdown.org/yihui/blogdown/html.html)
    - [other-themes](https://bookdown.org/yihui/blogdown)