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

`docker container prune` : remove **todos** os containers parados de uma vez (libera espaço da writable layer)
  - sem flags : pede confirmação e apaga todos com status `Exited` (e outros não running)
  - `-f` / `--force` : não pede confirmação
  - `--filter` : limita o que remove (ex.: `until=24h`, `label=...`)

*ATENÇÃO:* é a “faxina em massa” dos parados — equivalente a vários `docker rm`, não a um `rm` pontual. **Não** mexe em containers `Up`. **Não** remove imagens nem volumes **nomeados**; volumes **anônimos** órfãos podem sobrar (aí entra `docker volume prune`). Confira antes com `docker ps -a` / `docker ps -a -s`. Não confunda com `docker image prune` nem com `docker system prune` (este último é bem mais amplo).

`docker image` : gerencia imagens (grupo de subcomandos)
  - `ls` / `docker images` : lista imagens locais
  - `pull` : baixa uma imagem do registry (ver abaixo)
  - `rm` / `docker rmi` : remove uma ou mais imagens (ver abaixo)
  - `inspect` : mostra detalhes em JSON da imagem
  - `history` : mostra as camadas/histórico da imagem
  - `tag` : cria uma nova tag para uma imagem existente
  - `build` : constrói imagem a partir de um Dockerfile (ver abaixo)
  - `prune` : remove imagens não usadas (ver abaixo)

`docker build` / `docker image build` : constrói uma **imagem** a partir de um `Dockerfile` e de um **contexto** (pasta enviada ao daemon)
  - forma típica: `docker build -t nome:tag .` (o `.` = contexto = diretório atual)
  - `-t` / `--tag` : nomeia a imagem (`repositório:tag`); repositório em **minúsculas**
  - `-f` / `--file` : caminho do Dockerfile se não for `./Dockerfile`
  - `--no-cache` : ignora cache de camadas (build “do zero”)
  - o contexto inclui os arquivos que o `COPY`/`ADD` pode enxergar (por isso se roda o build **de dentro** da pasta do app, ou se passa o path certo)

*ATENÇÃO:* `build` gera **imagem**, não sobe container — depois vem `docker run`. A tag do `-t` (`imgexec7:1.0`) é o nome local da imagem nova; **maiúsculas no nome do repositório são rejeitadas** (`imgExec7:1.0` → erro *repository name must be lowercase*). O `.` no final importa: é o contexto, não “enfeite”. Cada instrução do Dockerfile vira camada; mudar só o `index.html` reaproveita cache do `FROM` e refaz o `COPY`. `build` **não** altera containers que já estavam rodando com a imagem antiga — é preciso novo `run` (ou recriar o container) para servir a imagem recém-buildada.

`docker pull` / `docker image pull` : baixa (ou atualiza) uma imagem de um registry — no dia a dia, em geral o **Docker Hub**
  - forma típica: `docker pull nome:tag` (ex.: `python:3.13-alpine3.23`)
  - sem tag explícita: assume **`latest`** (ex.: `docker pull nginx` = `nginx:latest`)
  - imagens oficiais do Hub: `docker.io/library/<nome>:<tag>` (o CLI aceita só `nome:tag`)
  - `-q` / `--quiet` : saída mínima
  - `--platform` : escolhe plataforma (ex.: `linux/amd64`) quando a imagem é multi-arch

*ATENÇÃO:* a **tag** não é “detalhe cosmético” — ela escolhe **qual** build você baixa (`3.13-alpine3.23` ≠ `3.12` ≠ `latest`). `latest` muda com o tempo no registry; em estudo/reprodutibilidade, prefira tag **específica**. `pull` só coloca a imagem no host; **não** cria container (isso é `run`). Se a imagem já existe, um novo `pull` pode baixar camadas mais novas da **mesma** tag. `run` sem a imagem local também puxa automaticamente (como no lab do Nginx); `pull` explícito deixa o download separado e visível.

`docker rmi` / `docker image rm` : remove uma ou mais **imagens** locais
  - aceita **nome:tag** (ex.: `ubuntu:22.04`) ou **ID** (prefixo curto ok)
  - vários alvos no mesmo comando: `docker rmi img1 img2`
  - `-f` / `--force` : força a remoção (ex.: imagem ainda referenciada de forma “incômoda”, ou para destravar casos comuns de estudo)
  - `--no-prune` : não apaga as camadas/pais sem tag que ficariam órfãos após o `rmi`

*ATENÇÃO:* `rmi` é de **imagem**; `rm` é de **container** — não misture. Se algum container (mesmo `Exited`) ainda usa a imagem, o Docker **recusa** o `rmi` até você remover esses containers (`docker rm` / `container prune`). Remover uma **tag** de uma imagem que tem várias tags só “desetiqueta”; a imagem só some de fato quando não resta referência. Imagens dangling (`<none>`) e lixo em massa: prefira `docker image prune` (e `-a` com cuidado). `rmi` não remove containers nem volumes.

`docker image prune` : remove imagens não utilizadas (libera espaço)
  - sem flags : remove só imagens dangling (sem tag, `<none>`)
  - `-a` / `--all` : remove todas as imagens não usadas por algum container
  - `-f` / `--force` : não pede confirmação
  - `--filter` : filtra o que remover (ex.: `until=24h`)
