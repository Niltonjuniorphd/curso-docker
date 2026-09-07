# Comandos Docker estudados

Formato: `<comando> : <o que ele faz> <flags>`

---

`docker ps` : lista containers em execução (só os que estão rodando agora)
  - `-a` / `--all` : inclui containers parados (exited)
  - `-q` / `--quiet` : mostra só os IDs
  - `-s` / `--size` : mostra o tamanho do sistema de arquivos do container
  - `-n` / `--last` : mostra os N containers criados mais recentemente (ex.: `-n 5`)
  - `-l` / `--latest` : mostra só o último container criado
  - `--filter` / `-f` : filtra por critério (ex.: `status=exited`, `name=meuapp`, `ancestor=ubuntu`)
  - `--format` : formata a saída (ex.: tabela customizada)
  - `--no-trunc` : não corta IDs, comandos e nomes longos

*ATENÇÃO:* containers **parados ainda ocupam espaço** na máquina (metadados e, principalmente, a camada gravável com o que foi escrito dentro do container). `docker ps` sem `-a` esconde esses containers; use `docker ps -a` e `docker ps -a -s` / `docker system df` para ver o impacto. Limpe o que não precisar com `docker rm` ou `docker container prune` (volumes nomeados não são apagados só por remover o container).

`docker run` : cria e inicia um container a partir de uma imagem
  - `-d` / `--detach` : roda em segundo plano (detached)
  - `-it` : terminal interativo (`-i` mantém STDIN; `-t` aloca TTY) — comum para shell
  - `--name` : define o nome do container
  - `-p` / `--publish` : mapeia porta do host para a do container (ex.: `-p 8080:80`)
  - `-e` / `--env` : define variável de ambiente
  - `-v` / `--volume` : monta volume ou bind mount
  - `--rm` : remove o container automaticamente ao sair
  - `--network` : conecta o container a uma rede
  - imagem e comando opcional no final (ex.: `docker run -it ubuntu bash`)

*ATENÇÃO:* `docker run` **sempre cria um container novo**. Não “reabre” um que já existia. Consequências: cada `run` gera outro ID/nome (ou erro se `--name` repetir); containers parados vão acumulando (`docker ps -a`) e ocupam espaço/histórico até você remover (`docker rm`) ou usar `--rm`; para só ligar de novo um container já criado, use `docker start` (e `docker start -ai` se for interativo), não outro `run`.

`docker start` : inicia um ou mais containers que já existem (parados)
  - `-a` / `--attach` : anexa a saída do processo principal ao terminal
  - `-i` / `--interactive` : anexa o STDIN (com `-a`, uso típico interativo: `-ai`)

*ATENÇÃO:* `docker start` sem `-ai` sobe o container em background: fica `Up` no `docker ps`, mas você **não** ganha shell. Para entrar num container já `Up`, use `docker exec -it` (ou `attach` no processo principal).

`docker exec` : executa um comando **dentro** de um container que já está em execução
  - `-it` : interativo + TTY (ex.: `docker exec -it nome bash`)
  - `-u` / `--user` : roda como outro usuário
  - `-w` / `--workdir` : define o diretório de trabalho
  - `-e` / `--env` : variável de ambiente só para esse comando

*ATENÇÃO:* `exec` abre um processo **extra**. Dar `exit` no shell do `exec` **não** para o container — o processo principal continua. O container precisa estar `Up` (`start` antes, se estiver parado).

`docker stop` : para containers em execução de forma “educada” (SIGTERM; depois SIGKILL se não encerrar a tempo)
  - `-t` / `--time` : segundos de espera antes do kill forçado (padrão em geral 10s)

*ATENÇÃO:* `stop` encerra o processo principal → status `Exited`. Código `(137)` costuma ser 128+9 (SIGKILL), comum quando o processo não termina no prazo. Para forçar na hora: `docker kill`. Container parado continua existindo até `docker rm` (ainda ocupa a writable layer).

`docker attach` : conecta o terminal ao processo principal de um container já `Up`
  - detach típico sem matar o processo: `Ctrl+P` então `Ctrl+Q`

*ATENÇÃO:* diferente do `exec`: você cola no PID 1. `Ctrl+C` pode encerrar o processo principal e parar o container. No dia a dia de estudo, `exec -it` costuma ser mais seguro.

`docker rm` : remove um ou mais containers (apaga o registro e a camada gravável)
  - aceita **nome** ou **ID** (ID pode ser prefixo curto, ex.: `1b49` de `1b49f38f8431`)
  - `-f` / `--force` : remove mesmo se estiver rodando (força a parada)
  - `-v` : remove volumes anônimos associados ao container

*ATENÇÃO:* por padrão só remove containers **parados**. Se estiver `Up`, faça `docker stop` antes (ou use `-f`). `rm` não apaga a **imagem**; volumes **nomeados** em geral permanecem. Depois do `rm`, aquele container (e arquivos tipo `filetest.txt` na writable layer) não existem mais — `start` nele falha; só um novo `run` cria outro.

`docker image` : gerencia imagens (grupo de subcomandos)
  - `ls` / `docker images` : lista imagens locais
  - `pull` : baixa uma imagem do registry
  - `rm` / `docker rmi` : remove uma ou mais imagens
  - `inspect` : mostra detalhes em JSON da imagem
  - `history` : mostra as camadas/histórico da imagem
  - `tag` : cria uma nova tag para uma imagem existente
  - `build` : constrói imagem a partir de um Dockerfile
  - `prune` : remove imagens não usadas (ver abaixo)

`docker image prune` : remove imagens não utilizadas (libera espaço)
  - sem flags : remove só imagens dangling (sem tag, `<none>`)
  - `-a` / `--all` : remove todas as imagens não usadas por algum container
  - `-f` / `--force` : não pede confirmação
  - `--filter` : filtra o que remover (ex.: `until=24h`)
