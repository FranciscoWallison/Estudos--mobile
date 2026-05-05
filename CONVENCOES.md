# Convenções do projeto

Como manter o repositório consistente conforme ele cresce. Tudo aqui é para ser lido em 5 minutos.

## Estrutura

```
ferramentas/<categoria>/<ferramenta>.md   # 1 ferramenta = 1 arquivo
dicas/<verbo-tarefa>.md                   # 1 fluxo end-to-end = 1 arquivo
modelos/                                  # templates copiáveis
```

- **Ferramenta** descreve a ferramenta em si (instalação, uso, dicas).
- **Fluxo (dicas/)** descreve uma tarefa end-to-end (ex: "capturar tráfego HTTPS"). Pode combinar várias ferramentas.

Se um conteúdo está duplicado entre uma ferramenta e um fluxo, mantém no fluxo e linka da ferramenta.

## Nomenclatura

- **Pastas e arquivos:** `kebab-case` minúsculo. Sem espaço, sem underscore, sem maiúscula.
  - `jadx.md`, não `JADX.md` nem `jadx_gui.md`.
  - `adb-proxy-global.md`, não `adb_rede.md`.
- **Fluxos** começam com verbo no infinitivo: `extrair-apk-de-app-instalado.md`, `capturar-trafego-https-android.md`.
- **Idioma:** PT-BR no conteúdo. Comandos e nomes próprios em inglês quando for o original.

## Cabeçalho padrão

Todo arquivo de **ferramenta** começa com o template em [modelos/ferramenta-template.md](modelos/ferramenta-template.md).
Todo arquivo em **dicas/** começa com o template em [modelos/fluxo-template.md](modelos/fluxo-template.md).

O bloco de citação inicial (`> **Categoria:**`, etc.) é obrigatório — é ele que torna a ferramenta filtrável visualmente quando você abre o arquivo.

## Blocos de código

Sempre com linguagem declarada:

```bash
adb devices -l
```

```powershell
adb reverse tcp:8092 tcp:52714
```

```js
const http = require('http');
```

Nunca cole comandos sem bloco — quebra o copy-paste.

## Links

- **Internos:** sempre relativos, no formato `[texto](caminho/arquivo.md)`. Nunca caminho absoluto.
- **Entre ferramentas:** `[adb](../dispositivo-adb/adb.md)`.
- **De ferramenta para fluxo:** `[fluxo X](../../dicas/x.md)`.
- **Externos:** só quando agregar (vídeo demonstrando, doc oficial). Evite linkar Stack Overflow.

## Adicionando coisas novas

### Nova ferramenta

1. Identifique a categoria. Se não existir, pode criar (ver abaixo).
2. Copie [modelos/ferramenta-template.md](modelos/ferramenta-template.md) pra `ferramentas/<categoria>/<nome>.md`.
3. Preencha. Foque em "como usar no Windows" e em "pegadinhas".
4. Adicione uma linha no `ferramentas/<categoria>/README.md` (tabela).
5. Se a ferramenta vira parte de um fluxo, linke em `Ver também`.

### Nova categoria

1. Crie a pasta `ferramentas/<categoria>/`.
2. Crie `ferramentas/<categoria>/README.md` com: 1 parágrafo sobre o que entra ali + tabela vazia (vai sendo preenchida).
3. Adicione uma linha no `README.md` raiz na lista de categorias.

### Novo fluxo (dicas/)

1. Copie [modelos/fluxo-template.md](modelos/fluxo-template.md) pra `dicas/<verbo-tarefa>.md`.
2. Preencha. Cada passo precisa ter comando concreto e o que conferir.
3. Adicione uma linha no `dicas/README.md`.
4. Em cada ferramenta envolvida, adicione o link na seção `Ver também`.

## O que NÃO incluir

- Conteúdo desatualizado/proprietário (URLs de APIs internas, tokens, dados de cliente).
- Procedimentos para uso ilegal. Engenharia reversa é pra estudo, malware analysis ou auditoria de apps que você tem direito.
- Tutorial gigante de algo que tem doc oficial boa — prefira linkar.
