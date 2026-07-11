# Desafio Técnico de Data Science — Digital Grid

**Vaga:** Pesquisador em Ciência de Dados — P&D Inteligência Energética (Bolsista)
**Esforço estimado:** 3 horas · **Prazo de entrega:** 5 dias corridos

> **Leia antes de começar:** não esperamos que todos concluam tudo. **Preferimos três
> questões bem-feitas e criticadas a seis questões corridas.** Se você tiver que
> escolher, escolha profundidade. E se você discordar de algo no enunciado, diga —
> discordância fundamentada conta a favor.

---

## Contexto: Geração Distribuída e o rateio de créditos

No Brasil, o crescimento da Geração Distribuída (GD) trouxe o **Sistema de Compensação
de Energia Elétrica (SCEE)**. A energia excedente gerada por uma usina é injetada na
rede e convertida em **créditos**, que abatem o consumo futuro de múltiplas **Unidades
Consumidoras (UCs)**.

O desafio operacional é o **rateio**: decidir que percentual da geração vai para cada
UC. Historicamente isso é feito de forma **estática** — define-se um percentual fixo
por UC no início do contrato e quase nunca se revisa. O resultado são duas patologias:

- **Vacância** — crédito acumulado em UCs que não vão usá-lo (o cliente mudou de
  endereço, reduziu o consumo, ou simplesmente cancelou e ninguém tirou da lista).
  Energia limpa gerada e desperdiçada.
- **Overbooking** — UCs que cresceram e ficaram descobertas, tendo que comprar energia
  cara da distribuidora enquanto sobra crédito parado em outro lugar da carteira.

A Digital Grid resolve isso com rateio dinâmico e previsão. Este desafio é uma versão
reduzida — e real — do que fazemos.

---

## Dados

Dois arquivos, na raiz deste repositório. O histórico vai de **julho/2023 a
junho/2026** (36 meses).

**`consumer_unit_data.xlsx`** — histórico mensal das UCs atreladas à usina.

| Coluna | Descrição |
| --- | --- |
| `Geração Mensal Referência Month` | mês de referência |
| `Unidade Consumidora (UC) Número de Instalação` | identificador da UC |
| `Conta Consumo (kWh)` | energia consumida no mês (**Ec**) |
| `Conta Saldo Acumulado (kWh)` | crédito acumulado **antes** do rateio do mês (**Cr**) |

**`power_plant_data.xlsx`** — histórico mensal de geração da usina.

| Coluna | Descrição |
| --- | --- |
| `Geração Mensal Referência Month` | mês de referência |
| `Unidade Consumidora (UC) Usina (Nickname)` | nome da usina |
| `Geração Mensal SUM Energia Gerada (kWh)` | energia gerada no mês (**G**) |

São dados de produção, com todos os defeitos que dados de produção têm. **Não fizemos
nenhuma limpeza para você.**

---

## Notação

| Símbolo | Significado |
| --- | --- |
| `Ec` | Energia consumida (kWh) |
| `OpF` | Quota de perfil — use `1.0` (100%) |
| `ECA` | Energia consumida aparente = `Ec × OpF` |
| `D` | Custo de disponibilidade — use `0` |
| `Cr` | Crédito acumulado na UC antes do rateio |
| `Co` | Cobertura — necessidade real de injeção de novos créditos |
| `P` | Percentual de rateio alocado para a UC |
| `G` | Geração total da usina |

**Mês de referência para todo o desafio: `2026-06-01`.**

---

## Q0 — Contrato de dados *(~30 min)*

Antes de qualquer modelo, construa a fundação.

Escreva uma função `carregar_e_limpar()` que leia os dois arquivos e devolva os dados
prontos para análise. Ela deve ser **idempotente** (rodar duas vezes dá o mesmo
resultado) e deve produzir um **relatório de qualidade**: uma tabela com quantos
registros entraram, quantos foram descartados ou corrigidos, e **por quê**.

Documente cada decisão de limpeza. Não existe resposta única — existe decisão
justificada. O que não aceitamos é limpeza silenciosa.

> Esta questão vale mais do que parece. Tudo que vem depois depende dela.

---

## Q1 — Cobertura e rateio *(~25 min)*

Implemente a lógica de alocação de créditos:

1. **Consumo aparente:** `ECA = Ec × OpF`
2. **Cobertura:** a UC só precisa de crédito novo se o consumo aparente superar o
   saldo que ela já tem.
   `Co = ECA − (Cr + D)` se `ECA > (Cr + D)`; caso contrário `Co = 0`.
3. **Percentual de rateio:** `P = Co / Σ(Co de todas as UCs da usina)`

Sua função deve **receber o mês como parâmetro** e funcionar para qualquer mês do
histórico — não apenas o de referência.

**Valide** que `Σ P = 1`.

**Responda no notebook:** o que acontece matematicamente com `P` se `Σ Co = 0`, ou
seja, se nenhuma UC precisar de crédito naquele mês? Como você trataria isso em
produção — e o que a usina faz com a energia que gerou?

---

## Q2 — SQL *(~20 min)*

Carregue os dados **limpos** da Q0 em um banco (SQLite, DuckDB — sua escolha) e
escreva **uma única query** que retorne, para o mês de referência:

```
uc | eca | cr | co | p
```

com `Co` e `P` calculados **em SQL**, não em Python. O resultado deve bater com o da
Q1.

---

## Q3 — Previsão de consumo *(~50 min)*

Preveja o `Conta Consumo (kWh)` de **2026-07-01** para as **Top 10 UCs** por consumo
histórico.

- **Split:** treine com dados até `2026-05-01` e valide em `2026-06-01`. Se conseguir,
  faça também uma **validação walk-forward** nos últimos meses — um único mês de teste
  é uma amostra de tamanho 1.
- **Modelos:** um *baseline* estatístico simples e um modelo mais avançado.
  **Justifique a escolha do modelo à luz do tamanho e da estrutura dos dados** — não
  há prêmio por usar a ferramenta mais sofisticada, há prêmio por usar a adequada.
- **Métricas:** MAPE e MAE. Se alguma delas se comportar mal com estes dados, diga
  isso e proponha alternativa.
- **Compare** baseline vs. modelo avançado e explique **por que** um superou o outro.
  Se o baseline venceu, diga que venceu.
- Lembre-se: você validou em junho, mas o pedido é prever **julho**.

---

## Q4 — Rebalanceamento *(~35 min)*

A Digital Grid tem um sistema chamado **REBALANCE**, que redistribui energia para
evitar vacância e overbooking. Simule uma versão simplificada.

**Cenário:** a usina fechou o mês de referência com **10.000 kWh excedentes**, acima
da soma das coberturas `Co` calculadas na Q1.

Escreva uma função que distribua esse excedente entre as UCs. Requisitos:

- **Não distribua igualmente.** Priorize as UCs que trazem mais segurança ao negócio,
  e explique o critério.
- **Restrição obrigatória:** nenhuma UC pode terminar com saldo acima de **3 meses**
  do seu consumo médio. Crédito parado é crédito desperdiçado.
- Se a Q3 ficou pronta, use a previsão para decidir. Se não, use o histórico.

**Demonstre** em um DataFrame como ficou o saldo de cada UC antes e depois da injeção.

---

## Q5 — Exploração e visão de negócio *(~20 min)*

**Qualidade:** que anomalias você encontrou nos dados? Como trataria cada uma numa
pipeline de produção (ETL)?

**Insights:** proponha **duas ações de negócio** concretas para a Digital Grid, com
base no que você viu. Ação de negócio é algo que alguém pode *fazer na segunda-feira* —
não é uma observação.

> Exemplo do formato esperado: *"20% das UCs concentram 80% do crédito ocioso.
> Recomendo um alerta automático que sinalize toda UC cujo saldo passe de N meses de
> consumo, para realocação."*

---

## Entrega

1. **Repositório Git** (ou `.zip`) com um **Jupyter Notebook** documentado, e um
   `requirements.txt` ou equivalente — precisamos conseguir rodar seu código.
2. **Vídeo de 3 a 5 minutos**, não listado (YouTube, Loom, Drive), apresentando a
   solução como se fosse para o CTO e o time de Produto:
   - **1 min** — o problema: o que é o rateio na GD, e a dor da vacância/overbooking.
   - **2 min** — a solução: qual modelo venceu na Q3 e **por quê**; como você pensou o
     rebalanceamento da Q4.
   - **1 min** — o insight mais interessante da Q5 e o que a Digital Grid faz com ele.

---

## Como avaliamos

| Peso | Critério |
| --- | --- |
| **35%** | **Qualidade de dados (Q0/Q5)** — as anomalias que você encontrou e como as tratou |
| **20%** | **Rigor na modelagem (Q3)** — o *porquê* por trás das métricas, não o número |
| **15%** | **Lógica de negócio (Q4)** — a priorização é defensável? conversa com a Q3? |
| **10%** | **Correção técnica (Q1/Q2)** — fórmulas certas, código vetorizado, em funções |
| **10%** | **Insight acionável (Q5)** |
| **10%** | **Comunicação (notebook + vídeo)** |

Boa sorte.
