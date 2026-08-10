# Teste 3 — Da geração suja ao rateio de créditos

Vaga: Pesquisador em Ciência de Dados — P&D Inteligência Energética (Bolsista)

**1h30, ao vivo, com a tela compartilhada.**

---

## O contexto, em um parágrafo

Uma usina fotovoltaica injeta energia na rede e essa energia vira **crédito** para abater o
consumo de várias Unidades Consumidoras (UCs) atreladas a ela. Quanto cada UC recebe é
decidido por um **rateio**: um percentual por UC, recalculado todo mês. Se o rateio for
mal feito, sobra crédito parado em quem não precisa (**vacância**) e falta em quem precisa
(**overbooking**). Para acertar o rateio de um mês futuro é preciso saber **quanto a usina
vai gerar** — e é aí que a previsão entra.

Você vai fazer o caminho inteiro: limpar a série da usina, prever a geração, e usar essa
previsão para ratear os créditos de **outubro/2026**.

---

## Os dados

**`power_plant_data_teste.xlsx`** — geração mensal da usina, **julho/2021 a junho/2026**
(60 meses). Vem de uma carteira real, anonimizada, **com os defeitos que dados de produção
têm**. Não fizemos limpeza para você.

**`consumo_medio_ucs.xlsx`** — as **10 UCs** atreladas a essa usina, com o **consumo médio
mensal** e o **saldo acumulado de créditos** de cada uma. Esta base já vem limpa: use os
valores como estão.

---

## Notação

| Símbolo | Significado |
| --- | --- |
| `Ec` | Consumo médio da UC (kWh) |
| `OpF` | Quota de perfil — use `1.0` (100%) |
| `ECA` | Energia consumida aparente = `Ec × OpF` |
| `D` | Custo de disponibilidade — use `0` |
| `Cr` | Crédito acumulado na UC antes do rateio |
| `Co` | Cobertura — necessidade real de injeção de novos créditos |
| `P` | Percentual de rateio alocado para a UC |
| `G` | Geração total da usina no mês |

**Mês de referência do rateio: `2026-10-01`.**

---

## Parte 1 — Olhe a série antes de modelar

Analise os dados da usina e **identifique os pontos outliers**.

Diga **quantos** são, **quais** são e **por que** você chamou cada um de outlier. Qual foi a
sua régua? Um mês fraco de chuva e uma falha de inversor aparecem do mesmo jeito num
gráfico — a diferença está no critério, e o critério é seu.

Antes de qualquer estatística: **olhe o arquivo**. Nem todo defeito de dado é um outlier.

---

## Parte 2 — Limpe

Escreva a limpeza como uma **função idempotente** (rodar duas vezes dá o mesmo resultado):

1. **Elimine a geração 0.** Mês com geração zerada não é geração baixa: é ausência de dado.
2. **Impute os outliers pela média.** Substitua cada ponto anômalo por uma média — qual
   média (vizinhos? mesmo mês de outros anos?) é decisão sua, **diga qual e por quê**.

Mostre um **antes e depois**: quantos registros entraram, quantos saíram, quantos foram
corrigidos e o valor de cada correção.

---

## Parte 3 — Preveja

**Quanto a usina vai gerar em julho, agosto, setembro e outubro de 2026?**

- Escolha um modelo, **ajuste com todo o histórico limpo** e entregue os **quatro números**.
- Construa também um **baseline simples** e compare. Não há vantagem em usar a ferramenta
  mais sofisticada — há vantagem em usar a adequada e saber dizer qual é. **Se o baseline
  vencer, diga que venceu.**
- Entregue uma **faixa de incerteza**: quanto você confia nesses números, e como chegou
  nessa faixa.

Se você validar o modelo, diga **como** separou treino e teste. Ajustar na série inteira e
"validar" no fim dela é vazamento — se fizer isso, precisa saber que fez.

---

## Parte 4 — Cobertura e rateio

**(a) Cobertura.** Compare, mês a mês, a **geração prevista** com o **consumo médio total
das 10 UCs**. Em que mês a usina passa a cobrir a carteira? E no **acumulado** dos quatro
meses, cobre? Responda com números — e diga o que a Digital Grid deveria fazer a respeito.

**(b) Rateio de OUT/2026.** Implemente a lógica de alocação de créditos:

1. **Consumo aparente:** `ECA = Ec × OpF`
2. **Cobertura:** a UC só precisa de crédito novo se o consumo aparente superar o saldo que
   ela já tem.
   `Co = ECA − (Cr + D)` se `ECA > (Cr + D)`; caso contrário `Co = 0`.
3. **Percentual de rateio:** `P = Co / Σ(Co de todas as UCs da usina)`

Entregue uma tabela `uc | eca | cr | co | p` e o **crédito alocado** a cada UC
(`P × G`, com `G` = a sua previsão de out/2026).

**Valide** que `Σ P = 1` — e pergunte-se se essa validação prova mesmo que o seu rateio está
correto.

**Responda:** o que acontece matematicamente com `P` se `Σ Co = 0`, ou seja, se nenhuma UC
precisar de crédito naquele mês? Como você trataria isso em produção?

---

## Parte 5 — SQL

Carregue os dados em um banco (**SQLite, DuckDB — sua escolha**) e escreva **uma única
query** que retorne, para o mês de referência:

```
uc | eca | cr | co | p
```

com `Co` e `P` calculados **em SQL**, não em Python. O resultado deve bater com o da Parte 4.

---

## Regras

- **Compartilhamento de tela obrigatório, do início ao fim.** Compartilhe a tela inteira —
  não apenas a janela do notebook — logo nos primeiros minutos, antes de começar a
  trabalhar, e mantenha até o encerramento.
- **Sem assistentes de IA** (Claude, ChatGPT, Copilot etc). Feche as abas e extensões de IA.
- **Pode consultar o seu teste anterior** — notebook, código, anotações, documentação das
  bibliotecas, fóruns e outros meios que não sejam IA ou Copilotos.
- **Pense em voz alta.** Se travar, diga que travou: conta a favor, não contra.
- **Respostas mais do que perguntas será o critério principal de avaliação.** Sempre com o
  máximo de respostas possível e o mínimo de perguntas.
- Entrega: **respostas**. Se não der tempo de codar alguma parte, responda mesmo assim — o
  número dito em voz alta, com o raciocínio, vale quase tanto quanto o código.

> Precisando se comunicar durante o teste? Escreva para **lucas@dg.energy**.
