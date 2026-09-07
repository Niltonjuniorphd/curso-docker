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
