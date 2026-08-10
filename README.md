# Teste 1 — Rateio de Créditos de Energia

Vaga: Pesquisador em Ciência de Dados — P&D Inteligência Energética (Bolsista)

Duração máxima: 2 horas

---

## O contexto, em um parágrafo

Uma usina fotovoltaica injeta energia na rede e essa energia vira **crédito** para abater o
consumo de várias Unidades Consumidoras (UCs) atreladas a ela. Quanto cada UC recebe é
decidido por um **rateio**: um percentual por UC, recalculado todo mês. Se o rateio for
mal feito, sobra crédito parado em quem não precisa (**vacância**) e falta em quem precisa (**overbooking**). Para acertar o rateio de um mês futuro é preciso saber **quanto a usina vai gerar** — e é aí que a previsão entra.

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

**(a)** Plote a série inteira e descreva o **comportamento da usina** — com números.

Adjetivo sem quantidade não conta: se você disser que alguma coisa é alta, baixa, forte ou
fraca, diga **quanto**, **quando** e **comparado com o quê**.

**(b)** Analise os dados da usina e **identifique os pontos outliers**.

Diga **quantos** são, **quais** são e **por que** você chamou cada um de outlier. Qual foi
a sua régua?

> Antes de qualquer estatística, **abra o arquivo e olhe**. Nem todo defeito de dado é um
> outlier.

---

## Parte 2 — Limpe os dados

Escreva a limpeza como uma **função idempotente** (rodar duas vezes dá o mesmo resultado) dos pontos que você identificou como outliers ou falha de medição.

Mostre um **antes e depois**: quantos registros entraram, **quantos saíram e por quê**,
quantos foram corrigidos, qual a metodologia de correção e o valor de cada ponto depois de
corrigido.

---

## Parte 3 — Preveja

Divida a série limpa em três conjuntos:

| Conjunto | Para que serve |
| --- | --- |
| **Treino** | o modelo aprende os padrões aqui: ajusta pesos e parâmetros |
| **Validação** | comparar modelos e ajustar hiperparâmetros, sem tocar no teste |
| **Teste** | a nota final — o modelo nunca viu estes meses |

**Onde cortar é decisão sua.** Diga **quais meses** colocou em cada conjunto e
**por quê**. Lembre que aqui os conjuntos são **fatias no tempo, em ordem**.

**(a) Treine e escolha.** Ajuste no **treino** e compare na **validação** quantos modelos quiser. Escolha um e diga **por que** ele ganhou.

**(b) Dê a nota.** Reajuste o modelo escolhido com **treino + validação** e preveja o **teste**. Entregue **um número: o MAPE**. É o desempenho real do seu sistema.

> O teste se olha **uma vez**. Se você voltar e trocar de modelo porque o MAPE do teste não agradou, ele virou validação — e a nota perdeu o sentido.

**(c) Agora preveja o que ninguém sabe.** Reajuste com **todo** o histórico limpo (os três
conjuntos) e responda: **quanto a usina vai gerar em julho, agosto, setembro e outubro de
2026?**

> A base termina em **junho/2026** — julho também é previsão, não consulta.

---

## Parte 4 — Cobertura e rateio

**(a) Cobertura.** Compare, mês a mês, a **geração prevista** com o **consumo médio total
das 10 UCs**. Em que mês a usina passa a cobrir a carteira? E no **acumulado dos quatro
meses**, cobre? Responda com números — e diga o que a Digital Grid deveria fazer a
respeito.

**(b) Rateio de OUT/2026.** Implemente a lógica de alocação de créditos:

1. **Consumo aparente:** `ECA = Ec × OpF`
2. **Cobertura:** a UC só precisa de crédito novo se o consumo aparente superar o saldo que
   ela já tem.
   `Co = ECA − (Cr + D)` se `ECA > (Cr + D)`; caso contrário `Co = 0`.
3. **Percentual de rateio:** `P = Co / Σ(Co de todas as UCs da usina)`

Entregue uma tabela `uc | eca | cr | co | p` e o **crédito alocado** a cada UC
(`P × G`, com `G` = a sua previsão de out/2026).

**Valide** que `Σ P = 1`. Depois responda: **um rateio errado pode passar nessa
validação?** Se puder, mostre como — e diga que checagem você acrescentaria.

---

## Parte 5 — SQL

Carregue os dados em um banco (**SQLite, DuckDB — sua escolha**) e escreva **uma única query** que devolva o rateio de **out/2026**:

```
uc | eca | cr | co | p
```

com `Co` e `P` calculados **em SQL**, não em Python. O resultado deve bater com o da Parte 4.

---

## Entrega

### 1. O código, num fork deste repositório

Faça um **fork** deste repositório (botão *Fork*, no topo da página), trabalhe nele e
nos mande o link do seu fork. Não abra Pull Request — assim um candidato não vê a
solução do outro.

O fork deve conter:

- um **Jupyter Notebook (.ipynb)** documentado, com as respostas e o raciocínio;
- um **`requirements.txt`** (ou `environment.yml`, ou `pyproject.toml`) — precisamos
  conseguir rodar seu código;
- um **`SOLUCAO.md`** curto, ou uma seção inicial no notebook, dizendo o que você fez,
  o que deixou de fora e por quê;
- **o link do seu vídeo, no topo do `README.md` do fork** (veja o item 2).

**Commite ao longo do trabalho**, não tudo de uma vez no fim. O histórico faz parte da
entrega: queremos ver como você pensou, não só onde chegou.

### 2. Um vídeo de 3 a 5 minutos, no YouTube

Grave um vídeo de 3 a 5 minutos apresentando a solução

**Suba no YouTube como "Não listado"** (*Unlisted*) — não como "Privado", senão não
conseguimos abrir. Não listado significa que só quem tem o link acessa; o vídeo não
aparece em buscas nem no seu canal.

> No YouTube: **Enviar vídeo → Visibilidade → Não listado.**

**Cole o link no topo do `README.md` do seu fork**, assim:

```markdown
# Desafio Digital Grid — <seu nome>

🎥 **Vídeo de apresentação:** https://youtu.be/xxxxxxxxxxx

---

### Para onde enviar

Você só precisa do **link do seu fork** e submeter na pela ferramenta da aplicação do teste.

---

## Regras

- **Duração do teste:** 2h
- **Linguagem de programação:** Python e SQL

> Precisando se comunicar durante o teste? Escreva para **lucas@dg.energy**.
