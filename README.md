<div align="center">

<img src="assets/logo-ufc.svg" alt="Universidade Federal do Ceará" height="80">

# Pici Hunt

## **Trabalho de Sockets: Jogo em Rede sobre o Campus do Pici**

**Graduação — Redes de Computadores I (TI0145) · 2026.1**

Departamento de Engenharia de Teleinformática · Prof.ª Dr.ª Atslands Rego da Rocha

Alunas: Gabriela Bezerra Pereira e Larissa Vitória Santos Menezes

</div>

---

## Sobre o jogo

Pici Hunt é um jogo para dois jogadores em rede local, inspirado em Batalha Naval e ambientado no Campus do Pici da UFC. Cada jogador esconde três ovos em pontos reais do campus, como o DETI, o CT, o RU e a Lagoa, e precisa encontrar os ovos do adversário antes que o outro encontre os seus.

A comunicação entre os computadores é feita em tempo real por meio de sockets TCP, usando a API POSIX. A interface gráfica é construída com SFML.

## Como funciona

A partida passa por cinco fases.

| Fase | Descrição |
|------|-----------|
| **CONNECT** | O jogador 1 pressiona `H` para hospedar e aguarda. O jogador 2 pressiona `C`, digita o IP do host e confirma com Enter. |
| **PLACE** | Cada jogador clica em três pontos do mapa para esconder os ovos. Esses pontos ficam visíveis apenas para quem os colocou. |
| **WAITING** | Tela de espera animada enquanto o outro jogador termina de posicionar os seus ovos. |
| **BATTLE** | Os turnos se alternam. A cada jogada, o jogador clica em um ponto do mapa e escolhe o tipo de busca. |
| **GAMEOVER** | Quem encontrar todos os ovos do adversário primeiro vence. |

## Tipos de busca

| Tecla | Tipo | Raio | Disparos |
|-------|------|------|----------|
| `1` | Precisa | 20 px | 5 |
| `2` | Padrão | 45 px | 3 |
| `3` | Expandida | 80 px | 1 |

## Compilação

As dependências são um compilador com suporte a C++17 e a biblioteca SFML 2.x, com os módulos Graphics, Window, System e Audio.

No Ubuntu ou Debian, instale com:

```bash
sudo apt install g++ libsfml-dev
```

No Arch Linux, o pacote é `sfml`, e no macOS, a instalação pode ser feita com `brew install sfml`.

Para compilar a versão de release, recomendada para jogar em rede, use:

```bash
make
```

Para a versão de desenvolvimento, que permite o uso do endereço de loopback `127.0.0.1` em testes locais, use `make dev`. A lista completa de opções está disponível com `make help`.

## Execução

### Dois computadores na mesma rede

No computador do host, execute `./pici_hunt`, pressione `H` ou clique em Gabriela na tela inicial e aguarde a conexão. No computador do cliente, execute `./pici_hunt`, pressione `C` ou clique em Larissa, digite o IP do host e confirme com Enter.

O IP do host pode ser consultado com `ip addr show` no Linux, `ifconfig` no macOS ou `ipconfig` no Windows. Use o endereço da interface local, geralmente no formato `192.168.x.x` ou `10.x.x.x`.

### Teste local

Para testar em um único computador, compile com `make dev` e abra duas instâncias do programa. Na primeira, pressione `H`. Na segunda, pressione `C`, informe `127.0.0.1` e confirme com Enter.

```bash
make dev

# Terminal 1
./pici_hunt_dev    # pressione H

# Terminal 2
./pici_hunt_dev    # pressione C, digite 127.0.0.1 e confirme
```

## Estrutura do projeto

```text
computer-networking/
├── main.cpp        # Ponto de entrada, loop principal, renderização e eventos
├── game.hpp        # Estado do jogo, lógica de turnos e pontos do campus
├── net.hpp         # Camada de rede com sockets TCP POSIX não bloqueantes
├── Makefile        # Compilação nos modos release e dev
└── assets/
    ├── font.ttf        # Fonte principal
    ├── music.mp3       # Trilha sonora
    ├── click.wav       # Som de clique
    ├── winner.wav      # Som de vitória
    ├── gameover.wav    # Som de derrota
    ├── mapa_pici.png   # Mapa do Campus do Pici
    ├── logo.png        # Logo do jogo
    ├── gabriela.png    # Avatar do Player 1
    ├── larissa.png     # Avatar do Player 2
    ├── egg.png         # Sprite do ovo
    └── *.png           # Fotos dos pontos do campus
```

## Protocolo de rede

A comunicação utiliza TCP na porta `55000`, com pacotes binários de tamanho fixo definidos pela estrutura `Packet`. Cada pacote tem a seguinte forma:

```text
┌────────┬────────┬──────────────────────────────────┐
│  type  │ player │            payload               │
│ 1 byte │ 1 byte │         (union, variável)        │
└────────┴────────┴──────────────────────────────────┘
```

Existem quatro tipos de pacote. O `READY` indica que o jogador terminou de posicionar os ovos. O `ATTACK` é enviado pelo atacante e carrega as coordenadas e o raio da busca. O `RESULT` é a resposta do defensor, informando se a busca foi um erro, um acerto ou uma vitória, além da quantidade de ovos restantes. O `GAMEOVER` comunica o identificador do vencedor.

O início da batalha depende de um handshake. Cada jogador envia `READY` ao outro, e a transição para a fase BATTLE só ocorre quando ambos os pacotes tiverem sido recebidos.

```text
Player 1  ──── READY ────►  Player 2
Player 1  ◄─── READY ────   Player 2
          (ambos receberam READY → transição para BATTLE)
```

## Pontos do campus

O jogo utiliza 17 pontos reais do Campus do Pici. Cada um possui um ID numérico e uma sigla de identificação.

| ID | Nome | Sigla |
|----|------|-------|
| 1 | Açude | ACUDE |
| 2 | Biblioteca CT | BCCP |
| 3 | Educação Física | EF |
| 4 | RU Velho | RUV |
| 5 | ICA | ICA |
| 6 | RU Novo | RUN |
| 7 | Sistemas e Mídias Digitais | SMD |
| 8 | DETI | DETI |
| 9 | Bloco 707 | B707 |
| 10 | Centro de Tecnologia | CT |
| 11 | Cantina da Química | CANTQ |
| 12 | Centro de Ciências | CC |
| 13 | Ciências Agrárias | CCA |
| 14 | Entrada Humberto Monte | EHM |
| 15 | Entrada Mister Hull | EMH |
| 16 | Entrada Pernambuco | EP |
| 17 | Cantina da Física | CANTF |

## Tecnologias

O projeto foi desenvolvido em C++17, com SFML 2.x para janela, gráficos e áudio, sockets POSIX não bloqueantes (`O_NONBLOCK`, `MSG_NOSIGNAL` e `MSG_DONTWAIT`) para a comunicação em rede, e Make como sistema de build.