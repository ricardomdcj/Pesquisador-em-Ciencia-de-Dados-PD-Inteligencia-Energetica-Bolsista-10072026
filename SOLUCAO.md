# SOLUCAO.md

**Candidato:** Ricardo M. D. C. Junior

Tudo está no [`solucao.ipynb`](solucao.ipynb), com o raciocínio explicado em cada seção.

## Como rodar

```bash
pip install -r requirements.txt
jupyter notebook solucao.ipynb   # Run All
```

## Resumo rápido

- [x] Q0 — `carregar_e_limpar()` idempotente + relatório de qualidade. Os problemas
  mais sérios da base: mesma UC com dois formatos de ID, status CANCEL embutido no
  identificador, faturas duplicadas e outliers de dígito extra.
- [x] Q1/Q2 — rateio em função vetorizada, conferido contra a mesma conta em SQL (DuckDB).
- [x] Q3 — os baselines venceram os modelos avançados nas duas previsões (explico por
  quê no notebook). Resposta da pergunta final: a usina NÃO cobre a carteira em julho —
  cobertura de ~62%, déficit de ~247 mil kWh.
- [x] Q4 — ~735 mil kWh presos nas UCs canceladas; redistribuição proporcional ao
  consumo previsto, com o teto de 1 mês. Sobram ~91 mil kWh sem destino.
- [x] Q5 — duas ações: realocar o crédito ocioso (com alerta automático pra não voltar
  a acumular) e inspeção técnica da usina (geração caindo desde fev/2026, não é sazonal).

## O que eu faria com mais tempo

Backtesting rolling em vez de validar num mês só; intervalos de previsão em vez de
ponto; e testar modelos com regressores externos (irradiância) pra geração. Não usei
SARIMA/boosting porque com 35 meses de série e os baselines ganhando do Holt-Winters,
mais complexidade não se justificava.
