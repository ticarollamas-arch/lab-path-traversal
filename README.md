# Lab: Path Traversal (CWE-22)

> ⚠ **Em construção** — a documentação está pronta; o código está em migração e será publicado em breve.

Ambiente didático que sobe uma aplicação com uma falha real de travessia de diretório, demonstra a exploração e aplica a correção. Serve para treinar revisão de código e validar ferramentas de SAST.

[![Licença](https://img.shields.io/badge/licen%C3%A7a-MIT-green)](LICENSE)

> **Uso restrito ao laboratório.** A aplicação é vulnerável de propósito e roda isolada em container. Não exponha na internet nem aponte o exploit para sistemas de terceiros.

## Estrutura

```
app-vulneravel/     aplicação com a falha
exploit/            prova de conceito contra o container local
patch/              o diff que corrige
docs/analise.md     a explicação técnica
docker-compose.yml
```

## Rodando

```bash
docker compose up -d
python exploit/poc.py --alvo http://localhost:8080
```

## A falha

A aplicação recebe um caminho vindo do usuário e o concatena à raiz de arquivos antes de abrir. Sem normalização, `../` sobe fora do diretório previsto e alcança arquivos de configuração do servidor.

O ponto central: **normalizar não basta se você não verificar o resultado.** A correção resolve o caminho absoluto e confirma que ele continua dentro da raiz permitida.

## Antes e depois

```python
# vulnerável
caminho = os.path.join(RAIZ, entrada_usuario)
return open(caminho).read()

# corrigido
caminho = os.path.realpath(os.path.join(RAIZ, entrada_usuario))
if not caminho.startswith(os.path.realpath(RAIZ) + os.sep):
    raise PermissionError("caminho fora da raiz permitida")
return open(caminho).read()
```

## Detecção

O achado é reproduzido por [«link do seu scanner»] com a regra CWE-22. O lab funciona como caso de teste de regressão da ferramenta.

## Leitura relacionada

- CWE-22 — Improper Limitation of a Pathname to a Restricted Directory
- OWASP Top 10 — A01: Broken Access Control
