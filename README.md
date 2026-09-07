# curso-docker

Ambiente de **estudos práticos de Docker e Docker Compose** — caderno de comandos, labs numerados e (no futuro) pastas por tema com Dockerfile/Compose quando o exercício pedir.

Não é um produto de produção: o foco é clareza didática, experimentação no host e registrar o que foi aprendido.

## Conteúdo principal

| Arquivo | Para quê |
|---------|----------|
| `comandos.md` | Referência rápida dos comandos e flags já estudados (+ avisos `*ATENÇÃO*`) |
| `exercicios.md` | Labs na ordem de aprendizado (roteiro e/ou registro de sessão) |
| `AGENTS.md` | Regras do repositório e do papel da IA (professor verificador) |

## Como estudar

1. Pratique no terminal (`docker …`).
2. Anote comandos novos em `comandos.md`.
3. Registre ou siga labs em `exercicios.md`.
4. Use diretórios por tópico (`01-…`, `apps/`, etc.) quando o lab precisar de arquivos próprios.

## Pré-requisito

Docker Engine e Docker Compose no host Linux.

## Progresso atual (CLI)

- Criar containers (`run`), listar (`ps` / `-a` / `-s`)
- Reutilizar (`start`), acessar (`exec`), parar (`stop`), remover (`rm`)
- Noções de imagens (`image`, `image prune`)
