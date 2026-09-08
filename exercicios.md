# Exercícios Docker

## Exercício 0 — Criar e iniciar containers com `docker run`

### Objetivo
Usar `docker run` para criar **e** iniciar um container a partir de uma imagem, entender o fluxo imagem → container, o papel de `-it` / `--rm` / `--name`, e fixar que **cada** `run` gera um container **novo**.

### Pré-requisito
Imagens locais disponíveis (ex.: `hello-world`, `ubuntu`). Se faltar:

```bash
docker pull hello-world
docker pull ubuntu
```

### O que fazer

1. Rodar o container mais simples (cria, executa o comando padrão da imagem e encerra):

```bash
docker run hello-world
```

Observe a mensagem de sucesso no terminal. O processo do container termina na hora (`Exited`).

2. Conferir que o container **parou**, mas ainda existe:

```bash
docker ps -a
```

Deve aparecer uma linha com imagem `hello-world`, comando `/hello` e status `Exited (0)`.

3. Abrir um shell interativo numa imagem maior:

```bash
docker run -it --name estudo-ubuntu ubuntu bash
```

- `-i` : mantém STDIN aberto
- `-t` : aloca um TTY (terminal)
- `--name estudo-ubuntu` : nome fixo (senão o Docker inventa um nome aleatório)
- `ubuntu` : imagem
- `bash` : comando no lugar do padrão da imagem

Dentro do container, rode algo simples e saia:

```bash
whoami
hostname
exit
```

4. Listar de novo:

```bash
docker ps -a
```

O `estudo-ubuntu` deve estar `Exited`. Anote o `CONTAINER ID`.

5. (Opcional) Repetir o `run` **com o mesmo nome** e ver o erro:

```bash
docker run -it --name estudo-ubuntu ubuntu bash
```

Esperado: conflito de nome — o container antigo ainda existe. Isso reforça: `run` não reabre o anterior.

6. (Opcional) Rodar e **apagar ao sair**, para não acumular lixo de estudo:

```bash
docker run --rm -it ubuntu bash
```

Ao dar `exit`, o container some de `docker ps -a`.

7. (Opcional) Segundo `hello-world` — note **outro** ID/nome:

```bash
docker run hello-world
docker ps -a
```

### O que observar / aprender
- `docker run` = **criar** container + **start** na mesma tacada.
- Sem `-d`, o terminal fica “preso” ao processo principal do container até ele terminar (ou você sair do shell).
- `hello-world` é efêmero: roda, imprime, exit 0 — por isso quase sempre só aparece em `ps -a`.
- `-it` + `bash` é o padrão de lab para “entrar” num Linux mínimo.
- `--name` ajuda a achar o container depois; nome duplicado falha se o antigo não foi removido.
- `--rm` evita o acúmulo que o Exercício 1 explora com `ps -a -s`.
- Rodar `docker run` de novo **sempre** cria outro container (novo ID), mesmo com a mesma imagem.

### Critério de acerto
- `docker run hello-world` imprime a mensagem e deixa um `Exited` em `docker ps -a`.
- `docker run -it --name ... ubuntu bash` abre shell; após `exit`, o mesmo nome aparece parado com o **mesmo** ID daquela criação.
- Fica claro a diferença: próximo passo para **reutilizar** esse container é `docker start` (Exercício 1), não outro `run`.

### Ligação com o Exercício 1
Os vários `ubuntu` / `hello-world` em `Exited` do Exercício 1 são o rastro típico de vários `docker run` sem `--rm`. O Exercício 1 parte desse estado para ensinar `ps -a`, tamanho e `start`.

---

## Exercício 1 — Containers parados vs em execução (`ps` / `start`)

### Objetivo
Perceber que `docker ps` esconde containers parados, que eles ainda existem (e ocupam espaço), e que `docker start` reutiliza um container já criado em vez de criar outro com `docker run`.

### O que foi feito

1. Listar **todos** os containers (incluindo parados):

```bash
docker ps -a
```

Saída observada:

```text
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
a938e2c18edf   ubuntu        "bash"     23 minutes ago   Exited (0) 22 minutes ago             peaceful_cray
eb163c04e7e7   ubuntu        "bash"     28 minutes ago   Exited (0) 24 minutes ago             optimistic_fermat
a46934b4420b   ubuntu        "bash"     2 hours ago      Exited (130) 2 hours ago              funny_shaw
1b49f38f8431   hello-world   "/hello"   2 hours ago      Exited (0) 2 hours ago                distracted_satoshi
194f2e094801   hello-world   "/hello"   6 days ago       Exited (0) 6 days ago                 zen_chatelet
```

2. Ver o **tamanho** de cada container (camada gravável + virtual):

```bash
docker ps -a -s
```

Saída observada:

```text
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES                SIZE
a938e2c18edf   ubuntu        "bash"     28 minutes ago   Exited (0) 27 minutes ago             peaceful_cray        12.3kB (virtual 115MB)
eb163c04e7e7   ubuntu        "bash"     32 minutes ago   Exited (0) 29 minutes ago             optimistic_fermat    44MB (virtual 159MB)
a46934b4420b   ubuntu        "bash"     2 hours ago      Exited (130) 2 hours ago              funny_shaw           12.3kB (virtual 115MB)
1b49f38f8431   hello-world   "/hello"   2 hours ago      Exited (0) 2 hours ago                distracted_satoshi   4.1kB (virtual 49.2kB)
194f2e094801   hello-world   "/hello"   6 days ago       Exited (0) 6 days ago                 zen_chatelet         4.1kB (virtual 49.2kB)
```

3. Listar só os **em execução** — lista vazia:

```bash
docker ps
```

4. **Reiniciar** um container que já existia (não criar outro):

```bash
docker start peaceful_cray
```

5. Conferir que agora ele aparece em execução:

```bash
docker ps
```

Saída observada:

```text
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
a938e2c18edf   ubuntu    "bash"    33 minutes ago   Up 11 seconds             peaceful_cray
```

### O que observar / aprender
- `docker ps` ≠ `docker ps -a`: sem `-a`, containers `Exited` somem da lista, mas continuam no host.
- Coluna `SIZE` em `ps -a -s`: o primeiro valor é o que **este** container acrescentou (writable layer); `virtual` inclui as camadas da imagem.
- `optimistic_fermat` (~44MB) gravou bem mais que `peaceful_cray` / `funny_shaw` (~12.3kB) — mesmo imagem `ubuntu`, uso interno diferente.
- `Exited (0)` = saiu “ok”; `Exited (130)` costuma indicar interrupção (ex.: Ctrl+C).
- `docker start peaceful_cray` reutilizou o **mesmo** ID `a938e2c18edf` — não foi um `docker run` novo.
- Container com `COMMAND "bash"` em background pode ficar `Up` sem TTY anexado; para shell interativo de novo, o caminho típico é `docker start -ai nome` ou `docker exec -it nome bash` (quando já está Up).

### Critério de acerto
- Entender por que `docker ps` estava vazio e `docker ps -a` não.
- Conseguir colocar um container parado em `Up` com `start` e ver o mesmo `CONTAINER ID`.
- Relacionar `-s` com ocupação de espaço mesmo parado.

---

## Exercício 2 — Remover containers parados (`rm`)

### Objetivo
Apagar containers `Exited` que não servem mais, liberando nome/ID e a writable layer; ver que dá para usar nome ou ID curto.

### O que foi feito
1. `docker ps` → vazio (nada rodando).
2. `docker rm zen_chatelet` → removeu pelo **nome** (`hello-world` antigo).
3. `docker ps -a` → `zen_chatelet` sumiu; restaram os outros `Exited`.
4. `docker rm 1b49` → removeu `distracted_satoshi` pelo **prefixo do ID** (`1b49f38f8431`).
5. `docker ps -a` → só os três `ubuntu` parados.

### O que observar / aprender
- `stop` ≠ `rm`: parar deixa o container no histórico; `rm` apaga de vez.
- Identificar por nome ou ID (completo ou prefixo único).
- Só remove parado por padrão; em execução → `stop` antes (ou `rm -f`).
- A **imagem** `hello-world` continua no host; só os containers foram embora.

### Critério de acerto
- Após cada `rm`, o alvo some de `docker ps -a`.
- Entender nome vs ID curto.

### Ligação
Limpeza do acúmulo visto no Exercício 1. O Exercício 3 usa um container que ainda existia (`peaceful_cray`).

---

## Exercício 3 — Acessar container em execução e parar (`exec` / `stop`)

### Objetivo
Entender que `docker start` deixa o container `Up` sem terminal anexado; usar `exec -it` para entrar e `stop` para encerrar.

### O que foi feito
1. Com `peaceful_cray` já `Up`: `docker exec -it peaceful_cray bash` → shell como `root` no filesystem do Ubuntu.
2. Dentro: `touch filetest.txt` (arquivo na camada gravável do container).
3. `exit` → voltou ao host; container **continuou** `Up` (`docker ps -a`).
4. `docker stop peaceful_cray` → `Exited (137)`.

### O que observar / aprender
- `start` sem `-ai` = liga em background (sem acesso interativo).
- `exec -it` = processo **extra** no container que já roda; `exit` do exec **não** para o container.
- `stop` encerra o processo principal; `(137)` ≈ SIGKILL (128+9), comum após timeout do stop em containers “só bash”.
- Arquivo criado com `exec` persiste na writable layer enquanto o container existir (reaparece num próximo `start`/`exec`).

### Critério de acerto
- Entrar com `exec -it`, sair e ver o container ainda `Up`.
- Parar com `stop` e ver `Exited` em `ps -a`.

### Ligação
Parte do estado do Exercício 1 (`peaceful_cray` / `start`). Complementa: acesso e parada.

---

## Exercício 4 — Rodar Nginx com `run -it` (pull automático e logs no terminal)

### Objetivo
Ver o ciclo completo com uma imagem de **serviço** (`nginx`): pull sob demanda, container nomeado, processo principal em primeiro plano (`-it` sem `-d`), encerramento com Ctrl+C e o que sobra em `ps -a` / `image ls`.

### O que foi feito

1. Criar e iniciar o container (imagem ainda não existia localmente):

```bash
docker run -it --name servidorNginx nginx
```

Saída observada (resumida):

```text
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
...
Status: Downloaded newer image for nginx:latest
/docker-entrypoint.sh: Configuration complete; ready for start up
... nginx/1.31.5 ... start worker processes
```

O terminal ficou preso aos logs do Nginx (processo principal em foreground).

2. Encerrar com **Ctrl+C** no terminal anexado. O Nginx recebeu SIGINT, workers saíram com código 0 e o master encerrou de forma ordenada.

3. Conferir containers:

```bash
docker ps -a
```

```text
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS                      PORTS     NAMES
7f5185dcc877   nginx     "/docker-entrypoint.…"   About a minute ago   Exited (0) 13 seconds ago             servidorNginx
a938e2c18edf   ubuntu    "bash"                   2 hours ago          Exited (0) 45 minutes ago             peaceful_cray
```

```bash
docker ps
```

Lista vazia — nada `Up`.

4. Ver imagens locais:

```bash
docker image ls
```

```text
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest   5dd0d3e6e255       25.9kB         9.49kB
nginx:latest         05b8cb60c354        253MB         69.2MB    U
ubuntu:latest        2260313b31c8        160MB         45.3MB    U
```

(`U` = In Use: ainda há container referenciando a imagem, mesmo `Exited`.)

### O que observar / aprender
- Sem a imagem local, `docker run` faz **pull** de `nginx:latest` e só então cria/inicia o container.
- `-it` **sem** `-d`: o terminal anexa ao processo principal — no Nginx você **vê os logs** ao vivo; Ctrl+C manda sinal ao PID 1 e pode parar o container (aqui `Exited (0)`, shutdown limpo).
- Diferente do lab com `ubuntu bash`: o comando padrão do Nginx **não** é um shell; o entrypoint sobe o servidor. Para “entrar” depois com shell, o caminho seria `exec` com o container `Up` (e em geral sobe-se com `-d`).
- **Não** houve `-p`: o Nginx escutou **dentro** da rede do container; do host não ficou porta publicada (ok para só observar logs; para testar no browser faltaria algo como `-p 8080:80`).
- `--name servidorNginx` fixou o nome; o container parado **continua existindo** — a imagem `nginx` permanece no host (~253MB de disk usage na listagem) e aparece com `U` enquanto `servidorNginx` (ou outro) existir.
- `peaceful_cray` (`ubuntu`) ainda está no histórico do Exercício 3 — por isso `ubuntu` também marca `U`.

### Critério de acerto
- `run` puxou `nginx` na primeira vez e subiu o container nomeado `servidorNginx`.
- Ctrl+C encerrou o processo; `ps -a` mostra `Exited (0)`; `ps` sem `-a` fica vazio.
- `image ls` lista `nginx:latest` local; entende-se que **imagem ≠ container** e que `U` liga imagem a container ainda registrado.

### Ligação
Retoma `docker run` (Exercício 0) com imagem de serviço e reforça `ps` / `ps -a` (Exercício 1). O Exercício 5 reusa o nome `servidorNginx` com `-d` e `-p` para publicar a porta no host.

---

## Exercício 5 — Nginx em background com porta publicada (`-d` / `-p`)

### Objetivo
Subir o Nginx de forma típica de serviço: detached (`-d`), mapear porta do **host** para a do **container** (`-p`), liberar o nome com `rm` do container antigo e validar a página no browser (`localhost:8080`).

### O que foi feito

1. Estado inicial — só parados, nada em execução:

```bash
docker ps -a
```

```text
CONTAINER ID   IMAGE     COMMAND                  CREATED        STATUS                    PORTS     NAMES
7f5185dcc877   nginx     "/docker-entrypoint.…"   23 hours ago   Exited (0) 23 hours ago             servidorNginx
a938e2c18edf   ubuntu    "bash"                   25 hours ago   Exited (0) 24 hours ago             peaceful_cray
```

```bash
docker ps
```

Lista vazia.

2. Remover o container antigo para **liberar o nome** `servidorNginx` (a imagem `nginx` permanece):

```bash
docker rm servidorNginx
```

```text
servidorNginx
```

```bash
docker ps -a
```

```text
CONTAINER ID   IMAGE     COMMAND   CREATED        STATUS                    PORTS     NAMES
a938e2c18edf   ubuntu    "bash"    25 hours ago   Exited (0) 24 hours ago             peaceful_cray
```

3. Novo `run` em detached com publish de porta:

```bash
docker run -d -p 8080:80 --name servidorNginx nginx
```

```text
5e1983e447876aff6167ac6ca8289dab0d87104a142e4e699ae9ffe68e98cd34
```

(ID novo — outro container, não o `7f5185dcc877` do Exercício 4.)

4. Conferir execução e mapeamento:

```bash
docker ps
```

```text
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
5e1983e44787   nginx     "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   servidorNginx
```

5. No browser (ou `curl`): `http://localhost:8080` → página padrão **Welcome to nginx!**

### O que observar / aprender
- `-d` / `--detach`: sobe em segundo plano; o terminal **não** fica preso aos logs (contraste com o Exercício 4 e `-it`).
- **Ordem das portas em `-p`:** sempre **`host:container`** — **esquerda = host**, **direita = container**.
  - Aqui: `-p 8080:80` → você acessa **8080 no host**; o Nginx escuta **80 dentro** do container.
  - No `docker ps`: `0.0.0.0:8080->80/tcp` lê na mesma lógica (host → container).
  - Inverter (`80:8080`) quebraria o lab: o Nginx da imagem oficial escuta na **80**, não na 8080.
- Sem `-p` (Exercício 4) o serviço existia só na rede do container; com `-p` o Docker publica no host (IPv4 e, aqui, também IPv6 `[::]:8080`).
- `rm` do `Exited` libera o **nome**; `run` de novo com o mesmo `--name` cria **outro** ID. A imagem local já estava cached — sem pull demorado.
- Resposta em `localhost:8080` confirma o caminho: browser/host → porta 8080 do host → encaminha para 80 do container → Nginx.

### Critério de acerto
- Após `rm`, o nome `servidorNginx` some de `ps -a` e pode ser reutilizado.
- `docker ps` mostra `Up` com `8080->80/tcp`.
- `http://localhost:8080` exibe a página Welcome to nginx.
- Fica decorada a regra: **esquerda host, direita container**.

### Ligação
Fecha o ciclo do Exercício 4 (Nginx sem porta / foreground). O Exercício 6 separa o download de imagem (`docker pull`) com tag explícita, sem depender do pull implícito do `run`.

---

## Exercício 6 — Baixar imagem do Docker Hub (`pull` + tags)

Site: [https://hub.docker.com/](https://hub.docker.com/) (Docker Hub — registry público padrão usado pelo `docker pull` neste lab).

### Objetivo
Usar `docker pull` de forma explícita, entender o papel da **tag** (`nome:tag`) e criar um container a partir da imagem baixada — aqui Python 3.13 em base Alpine.

### O que foi feito

1. Conferir que o Nginx do Exercício 5 ainda estava no ar:

```bash
docker ps
```

```text
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
5e1983e44787   nginx     "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   servidorNginx
```

2. Baixar uma tag **específica** do Docker Hub (não `latest` genérico):

```bash
docker pull python:3.13-alpine3.23
```

Saída observada (resumida):

```text
3.13-alpine3.23: Pulling from library/python
...
Digest: sha256:75f27d686432419c9d42420b2b9ef605868c7a0682a6be10a6601fad46c2df01
Status: Downloaded newer image for python:3.13-alpine3.23
docker.io/library/python:3.13-alpine3.23
```

3. Listar imagens locais:

```bash
docker images
```

```text
IMAGE                    ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest       5dd0d3e6e255       25.9kB         9.49kB
nginx:latest             05b8cb60c354        253MB         69.2MB    U
python:3.13-alpine3.23   75f27d686432       79.2MB         20.1MB
ubuntu:latest            2260313b31c8        160MB         45.3MB    U
```

(`python` ainda **sem** `U` — só imagem, nenhum container usando.)

4. Subir um container interativo com o interpretador Python dessa imagem:

```bash
docker run -it --name python python:3.13-alpine3.23 python
```

Dentro do REPL:

```text
Python 3.13.15 ... on linux
>>> import sys
>>> print(sys.version)
3.13.15 (main, Sep  1 2026, 00:03:34) [GCC 15.2.0]
>>> exit()
```

(Houve um typo `eixt()` → `NameError`; `exit()` encerrou o REPL e o container.)

5. Estado final dos containers:

```bash
docker ps -a
```

```text
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS                      PORTS                                     NAMES
61014a9af6e7   python:3.13-alpine3.23   "python"                 2 minutes ago    Exited (0) 10 seconds ago                                             python
5e1983e44787   nginx                    "/docker-entrypoint.…"   35 minutes ago   Up 35 minutes               0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   servidorNginx
a938e2c18edf   ubuntu                   "bash"                   26 hours ago     Exited (0) 24 hours ago                                               peaceful_cray
```

### O que observar / aprender
- **`docker pull`** baixa a imagem para o host **sem** criar container. Depois o `run` só instancia.
- Formato **`repositório:tag`**: aqui `python` é o repositório (oficial `library/python` no Hub) e **`3.13-alpine3.23`** é a tag — versão do Python + base Alpine 3.23.
- A tag manda no conteúdo: outra tag = outra imagem (tamanho, libs, versão). `alpine` costuma ser bem menor que a variante “full” (neste lab ~79MB de disk usage vs centenas de MB típicos de imagens maiores).
- Sem `:tag` no pull/run, o default é **`latest`** — prático, mas **móvel** no registry; tag pinada (como `3.13-alpine3.23`) deixa o estudo mais previsível.
- A linha final `docker.io/library/python:3.13-alpine3.23` é o nome canônico no Hub; no CLI basta `python:3.13-alpine3.23`.
- No `run`, repetir a tag na imagem (`python:3.13-alpine3.23`) e o comando (`python`) abre o REPL; `-it` + `--name python` — o **nome do container** (`python`) não é a mesma coisa que o **nome da imagem**.
- Contraste com o Exercício 4: lá o `run nginx` fez pull **implícito** de `latest`; aqui o download foi **explícito** e com tag escolhida.
- `servidorNginx` seguiu `Up` o tempo todo — pull/run de outra imagem não derruba containers já em execução.

### Critério de acerto
- `docker pull python:3.13-alpine3.23` conclui com `Downloaded newer image` (ou Image is up to date se já tiver).
- `docker images` / `docker image ls` lista `python:3.13-alpine3.23` com a tag completa visível.
- `run -it ... python` abre o REPL na 3.13.x; após `exit()`, o container `python` aparece `Exited` em `ps -a`.
- Fica claro: **tag escolhe a variante**; `pull` ≠ `run`.

### Ligação
Complementa o pull implícito dos labs Nginx. Liga a `comandos.md` (`docker pull`, tags, `rmi`). O Exercício 7 usa `exec` num Nginx para achar a pasta padrão do HTML e montar o primeiro `Dockerfile`.

---

## Exercício 7 — Pasta padrão do Nginx + primeiro `Dockerfile` (`COPY`)

### Objetivo
Descobrir **onde** a imagem oficial do Nginx serve o HTML estático (para não chutar o destino do `COPY`) e criar uma imagem própria em `exec7/` que substitui o `index.html` padrão.

### Dúvida que guia o lab
> “Tenho um container Nginx. Como **entrar com terminal** (não com logs) e ver em qual pasta devo colocar meu HTML?”

Resposta curta: `docker exec -it <container> bash` (ou `sh`) — **não** `docker logs` e **não** `attach`. A pasta padrão de conteúdo estático da imagem oficial é:

`/usr/share/nginx/html`

É esse caminho que entra no lado direito do `COPY` no Dockerfile.

### Pré-requisito / arquivos do lab
Pasta do exercício:

```text
exec7/
├── Dockerfile
└── index.html
```

`index.html` — página simples do lab (head + body).

`Dockerfile` criado neste exercício (instruções na **ordem** em que aparecem no arquivo):

```dockerfile
FROM nginx:1.28.2
COPY index.html /usr/share/nginx/html/index.html
```

#### 1. `FROM` (primeira linha — base da imagem)

```dockerfile
FROM nginx:1.28.2
```

- **O que é:** toda imagem construída por Dockerfile **começa** com `FROM`. Define a **imagem base** — o ponto de partida (filesystem, Nginx já instalado, paths padrão, entrypoint).
- **Sintaxe:** `FROM <repositório>:<tag>` — aqui `nginx` + tag **`1.28.2`** (versão pinada, mesma ideia do Exercício 6; não é `latest`).
- **Efeito prático:** sua imagem final = base Nginx 1.28.2 **mais** as camadas seguintes (`COPY`, etc.). Sem um `FROM` válido o build nem começa.
- **Por que Nginx:** o lab serve HTML estático; a base já traz o servidor e a pasta `/usr/share/nginx/html`.
- **Ordem:** `FROM` vem **antes** do `COPY` porque você só copia “para dentro” de algo que já existe na base.

#### 2. `COPY` (segunda linha — seu HTML na pasta padrão)

```dockerfile
COPY index.html /usr/share/nginx/html/index.html
```

- Destino = pasta descoberta com `exec` no container Nginx.
- Origem = arquivo no **contexto de build** (`exec7/`); destino = path **dentro da imagem**.

### O que foi feito (descoberta do path)

1. Nada em execução no momento da inspeção; religar o Nginx do Exercício 5:

```bash
docker ps
docker start servidorNginx
```

```text
servidorNginx
```

2. Abrir **shell** no container (terminal, não logs):

```bash
docker exec -it servidorNginx bash
```

Prompt observado: `root@5e1983e44787:/#`

3. Explorar o filesystem e ir à pasta de conteúdo do Nginx:

```bash
ls
cd /usr/share/nginx/html
ls
```

```text
50x.html  index.html
```

Aí estão as páginas padrão da imagem (`Welcome to nginx!` no `index.html` de fábrica).

4. Com o path confirmado, o `Dockerfile` em `exec7/` usa o mesmo destino no `COPY`, para **sobrescrever** esse `index.html` na **build** da imagem nova (não no container `servidorNginx` ao vivo).

### O que foi feito (build + run)

5. No diretório do lab:

```bash
cd ~/estudos/cfb-docker/exec7
ls   # Dockerfile  index.html
```

6. Primeira tentativa de build — **tag inválida** (maiúsculas no repositório):

```bash
docker build -t imgExec7:1.0 .
```

```text
ERROR: failed to build: invalid tag "imgExec7:1.0": repository name must be lowercase
```

7. Build correto (nome em minúsculas):

```bash
docker build -t imgexec7:1.0 .
```

Saída (resumida): pull/uso de `nginx:1.28.2` → camada `COPY index.html ...` → `naming to docker.io/library/imgexec7:1.0`.

8. Conferir a imagem local:

```bash
docker images
```

```text
IMAGE                    ID             DISK USAGE   CONTENT SIZE   EXTRA
...
imgexec7:1.0             1dbcb28975b9        237MB         62.9MB
nginx:latest             ...                                    U
...
```

9. Subir container da **imagem buildada** (não a `nginx` pura do lab 5):

```bash
docker run -d -p 8080:80 --name exec7-container imgexec7:1.0
docker ps
```

```text
CONTAINER ID   IMAGE          ...   PORTS                                     NAMES
e481758a1547   imgexec7:1.0   ...   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   exec7-container
```

10. Teste no browser em `http://localhost:8080`.

### Contratempo: cache do browser (hard refresh)

Na primeira olhada o site ainda parecia o **Welcome to nginx!** padrão — parecia que o `COPY`/build tinha falhado.

**Não era falha do Docker.** Checagem no host:

- `docker exec exec7-container cat /usr/share/nginx/html/index.html` → página **Olá do Exercício 7**
- `curl http://127.0.0.1:8080/` → a mesma página do lab

O browser tinha **cache** da visita anterior a `localhost:8080` (quando o `servidorNginx` do Exercício 5 servia o HTML de fábrica na **mesma URL/porta**).

**Correção:** hard refresh (`Ctrl+Shift+R` / `Ctrl+F5`), ou janela anônima, ou confiar no `curl` (sem cache de página). Depois do hard refresh → **Olá do Exercício 7**.

*ATENÇÃO (lab):* trocar a imagem atrás de `localhost:8080` **não** obriga o browser a buscar de novo. Em dúvida: `curl`, `exec cat` no `index.html`, depois hard refresh.

### O que observar / aprender
- **`logs`** = saída do processo; **`exec -it`** = shell extra no container `Up`. `exit` do exec **não** para o Nginx.
- **`attach`** cola no PID 1 (Nginx), não é o jeito certo de “abrir pasta e digitar `ls`”.
- A imagem oficial serve estáticos em **`/usr/share/nginx/html`**. Achar com `exec` evita `COPY` no path errado.
- Ordem no Dockerfile: **`FROM`** (base + tag pinada) → **`COPY`** (HTML do lab em cima do padrão).
- **`docker build -t nome:tag .`**: lê o Dockerfile, manda o **contexto** (`.`), gera imagem local. Não inicia container.
- Nome da imagem no `-t`: **só minúsculas** no repositório (`imgexec7:1.0`, não `imgExec7:1.0`).
- **`run` da imagem nova** (`imgexec7:1.0`) ≠ reutilizar o container `servidorNginx` (esse ainda é a imagem `nginx` “de fábrica”).
- Prova de verdade do conteúdo: `exec cat` / `curl`. Browser pode mentir por **cache** — hard refresh quando a porta/URL é a mesma de um lab anterior.

### Critério de acerto
- `exec -it` no Nginx de referência e listar `/usr/share/nginx/html`.
- Explicar **`FROM`** e **`COPY`** na ordem do Dockerfile.
- `docker build -t imgexec7:1.0 .` conclui; `docker images` lista `imgexec7:1.0`.
- `docker run -d -p 8080:80 --name exec7-container imgexec7:1.0` fica `Up` com `8080->80`.
- Conteúdo servido = página do lab (confirmado com `curl` e/ou browser após **hard refresh** se o Welcome antigo estiver em cache).

### Ligação
Une Exercício 3 (`exec`), 5 (Nginx + porta) e 6 (tags). Introduz **imagem derivada** (`Dockerfile` + `build` + `run`) em vez de só puxar pronta do Hub. Ver também `comandos.md` → `docker build`.
