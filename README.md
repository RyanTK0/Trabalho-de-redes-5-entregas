# Minha pequena internet

Laboratório do trabalho semestral de **Redes e Sistemas Distribuídos** —
Prof. Me. Luiz Ricardo Mantovani da Silva.

Uma topologia só, que cresce ao longo do semestre: começa com duas redes que
não se falam e termina com um serviço replicado, roteamento que se refaz
sozinho e a demonstração de por que uma senha não deve viajar sem proteção.

Tudo roda no **Google Cloud Shell**, que é gratuito, não pede cartão de crédito
e dá a cada aluno uma máquina Linux com root só dela.

---

## Começando

Construa a imagem do laboratório — uma vez por sessão do Cloud Shell:

```bash
make base
```

Suba a topologia da primeira entrega:

```bash
make up E=1
```

Rode as provas:

```bash
make verificar E=1
```

`make verificar` **começa vermelho de propósito**. O laboratório vem
incompleto: o roteiro de verificação é o enunciado de verdade, e o trabalho é
levá-lo ao verde.

Enunciado de cada entrega em [`entregas/`](entregas/).

---

## As cinco entregas

| # | Semana | Tema | Vale |
|---|---|---|---|
| [E1](entregas/E1.md) | 6 | Dois segmentos e um serviço | 0,8 |
| [E2](entregas/E2.md) | 9 | O roteador e o encapsulamento | 1,4 |
| [E3](entregas/E3.md) | 11 | O serviço não pode cair | 1,2 |
| [E4](entregas/E4.md) | 12 | A rota se refaz sozinha | 1,2 |
| — | 14 | Apresentação | 1,4 |
| [E5](entregas/E5.md) | 13 | O pacote entrega o segredo · **Cartilha** | **2,0 (Extensão)** |

E1–E4 e a apresentação compõem os 6,0 de avaliação prática. E5 é a atividade
de extensão, com nota escalonada em 1,2 / 1,7 / 2,0.

---

## Regras do jogo

**A entrega é o repositório, nunca o ambiente.** A correção é feita clonando o
repositório do grupo numa Cloud Shell limpa e rodando `make up` e
`make verificar`. Se não subir lá, não entregou.

**Evidência é obrigatória.** `make evidencias E=<n>` grava a saída da
verificação; os `.pcap` das entregas 2 e 5 vão junto. Isso é versionado — nunca
apaguem a pasta `evidencias/`.

**Imagem pequena.** A máquina do Cloud Shell é efêmera e recicla o cache de
imagens: só a pasta pessoal (5 GB) sobrevive. Por isso a base é Alpine.

**Nada de varredura de rede.** Os termos de uso do Cloud Shell proíbem
explicitamente varredura, e a conta que descumprir é desligada. Todo o trabalho
aqui é captura passiva dentro da rede virtual do próprio grupo — o que é outra
coisa, e é permitido.

---

## Para o professor

Antes de liberar para a turma, uma vez, numa Cloud Shell limpa:

```bash
make autoteste
```

Confere as quatro coisas de que o laboratório depende e que documentação
nenhuma garante: criação de bridge com sub-rede própria, `tcpdump` com
`NET_RAW` efetivo, `net.ipv4.ip_forward` por contêiner, e o pacote `bird` no
Alpine (só a E4 depende dele).

### Botão "Abrir no Cloud Shell"

Para cada entrega, na página do trabalho:

```
https://shell.cloud.google.com/cloudshell/editor
  ?cloudshell_git_repo=https://github.com/LuizRMSilva1973/redes-lab
  &cloudshell_tutorial=entregas/E1.md
  &show=ide%2Cterminal
```

O aluno cai no terminal com o repositório clonado e o enunciado aberto como
roteiro guiado ao lado. Trocar `E1.md` pela entrega da vez.

O repositório precisa estar em **GitHub ou Bitbucket**, público.

### Correção

```bash
git clone <repo-do-grupo> && cd <repo> && make up E=2 && make verificar E=2
```

Os roteiros imprimem o valor observado em cada ponto, não só passou/falhou —
dá para corrigir lendo a saída.

-------------------------------------------------------------------------------

## Entrega 1 — Dois segmentos e um serviço

Nome                                     |   RA   |          E-mail          |
| -------------------------------------- | ----- -| ------------------------ |
| Ryan Taquita Konda                     |24110828|ryantaquitakonda@gmail.com|
| Daniel Shinji Onoue                    |24111099|  daniel.onoue@gmail.com  |
| Guilherme Henrique Carloni de Carvalho |24110810| gui.carloni21@gmail.com  |
| Henrique Augusto Forti                 |24110083|henrique.a.forti@gmail.com|
| Leonardo de Oliveira Prata             |24110804|leonardo.prata770@gmail.com|

**Disciplina:** Redes e Sistemas Distribuídos
**Professor:** Prof. Me. Luiz Ricardo Mantovani da Silva
**Trabalho:** Minha pequena Internet
**Repositório:** https://github.com/RyanTk0/Trabalho-de-redes-5-entregas-

### Plano de endereçamento

A topologia possui dois segmentos de rede independentes, cada um utilizando uma sub-rede IPv4 `/24`.

#### Segmento A

* Sub-rede: `10.0.10.0/24`
* Máscara: `255.255.255.0`
* Endereço de rede: `10.0.10.0`
* Broadcast: `10.0.10.255`
* Total de endereços: `2^8 = 256`
* Endereços utilizáveis: `256 - 2 = 254`

| Máquina   | Endereço IP     |
| --------- | -----------------
| `host-a1` | `10.0.10.10/24` |
| `host-a2` | `10.0.10.11/24` |
| `srv-a`   | `10.0.10.20/24` |

#### Segmento B

* Sub-rede: `10.0.20.0/24`
* Máscara: `255.255.255.0`
* Endereço de rede: `10.0.20.0`
* Broadcast: `10.0.20.255`
* Total de endereços: `2^8 = 256`
* Endereços utilizáveis: `256 - 2 = 254`

| Máquina   | Endereço IP     |
| --------- | --------------- |
| `host-b1` | `10.0.20.10/24` |
| `host-b2` | `10.0.20.11/24` |

### Isolamento entre os segmentos

Os segmentos A e B são redes IP diferentes: o segmento A utiliza `10.0.10.0/24`, enquanto o segmento B utiliza `10.0.20.0/24`.

Na Entrega 1 não existe um roteador conectando essas duas redes. Além disso, os hosts foram configurados sem rota padrão. Dessa forma, um host do segmento A possui uma rota diretamente conectada apenas para a rede `10.0.10.0/24`. Quando `host-a1`, por exemplo, tenta alcançar `10.0.20.10`, o destino está fora da sua rede local e não existe uma rota para encaminhar o pacote.

Por isso, o sistema retorna `Network is unreachable`. O mesmo princípio impede que os hosts do segmento B alcancem o servidor `srv-a` no segmento A.

Esse isolamento é confirmado pela verificação da Entrega 1: a comunicação funciona entre máquinas do mesmo segmento, mas falha entre os segmentos.
