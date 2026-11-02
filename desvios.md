# Desvios do protocolo v1

O protocolo congelado (release `protocolo-v1`, commit `dacba55`) não é editado. Toda mudança posterior fica registrada aqui, com data e motivo.

Estado em 2026-11-02: nenhum retorno calculado e nenhuma cota lida. Foram lidos apenas dias com informe, PL e cotistas (`dados/presenca_mensal.csv`). O valor de `VL_QUOTA` foi descartado na extração, guardando-se só se era válido.

## D1. BOVA11 sai do informe diário em 09/2018

**Achado:** a série do BOVA11 tem 35 dos 120 meses, de 11/2015 a 09/2018, com último informe em 06/09/2018. O fundo não foi cancelado.

**Causa:** não é específica do BOVA11. Os 15 fundos de tipo `FIIM` presentes em 09/2018 têm último informe em 06/09 ou 10/09/2018, e nenhum `FIIM` aparece em 10/2018. O motivo regulatório não foi confirmado.

**Por que o protocolo não cobre o caso:** a regra de encerramento trata de fundo encerrado, e o BOVA11 continua ativo. Com T = 35, o teste perderia a comparação com o paper (T = 120).

**Opções:**

| Opção | Série do passivo | A favor | Contra |
|---|---|---|---|
| A (escolhida) | BOVA11 pelo preço de fechamento da B3 (COTAHIST), em toda a janela | Mantém o fundo escolhido; fonte oficial; é o preço que o investidor obtém | Preço de mercado contra cota nos ativos; precisa confirmar que não houve provento nem desdobramento |
| B | Itaú Personnalité Index Ibovespa, já declarado como alternativo | 120 meses, mesma fonte e tipo de dado | Taxa cadastrada de 2,0% ao ano; compara fundos de custo parecido |

Nas duas opções, o outro passivo continua reportado como robustez.

**Validação da opção A, antes de calcular qualquer Sharpe:**

1. Usar uma única fonte em toda a janela, sem emendar cota e preço.
2. Medir, de 11/2015 a 08/2018, a correlação e o erro absoluto entre os retornos mensais por preço e por cota do BOVA11. Reportar só essas duas medidas, sem médias nem Sharpe.
3. Listar variações diárias acima de 15% em módulo no preço e checar eventos do fundo.
4. Medir, em toda a janela, a correlação e o erro absoluto entre os retornos mensais por preço e os do Ibovespa, que é um índice de retorno total. A cota da CVM só cobre até 2018, e esta medida cobre os eventos posteriores. Reportar só essas duas medidas.

**Decisão (2026-11-02, antes de abrir qualquer retorno):** opção A. O passivo principal continua sendo o BOVA11, com retorno calculado pelo preço de fechamento da B3 em toda a janela. O Itaú Personnalité Index Ibovespa continua como robustez, pela cota da CVM.

**Série da B3, conferida sem ler preços (`dados/bova11_cotahist.csv`, manifesto em `dados/manifesto_cotahist.tsv`):**

- 2.504 pregões de 01/10/2015 a 31/10/2025, nos 121 meses, sem buraco maior que 5 dias corridos, sem data repetida e com pelo menos 13 negócios por dia.
- `TPMERC` = 010 em todas as linhas, `FATCOT` = 1 em todas, e um único ISIN (`BRBOVACTF003`).
- O preço usado é `PREULT`, o último negócio do pregão, e não um fechamento oficial.
- As linhas do COTAHIST não vêm em ordem de data dentro do arquivo (206 inversões); o CSV foi ordenado.
- Há 13 mudanças de distribuição (`DISMES`) na janela, 5 delas em 2025, cada uma seguida de cerca de 8 pregões com especificação `CI ATZ`. Os comunicados B3 035/2021-VNC e 164/2022-VNC descrevem as de 07/2021 e 12/2022 como procedimento interno, sem impacto financeiro nem de preço. O comunicado 085/2023-VNC anuncia a de 09/2023, e não li o efeito declarado nele. As outras 10 não foram conferidas, e não encontrei comunicado para as de 2025; o item 4 da validação cobre esse risco.

## Esclarecimentos de regra

Não mudam a seleção. Definem casos que o protocolo deixou em aberto, antes de abrir qualquer retorno.

- **C1. Fim da série:** usa-se o último informe, não a data de cancelamento. Em 9 fundos, o último informe vem mais de 60 dias depois da primeira data `CANCELADA`, e um deles tem duas datas de cancelamento no histórico.
- **C2. Cota inválida:** dia com `VL_QUOTA` vazia ou não positiva conta como dia sem informe. Afeta 233 fundos-mês, em 120 fundos e 2.420 dias.
- **C3. Buraco na série:** um único fundo tem buraco no meio (SAFRA ABSOLUTO 30, 2 meses em 2021, junto à data de cancelamento). Os meses ficam faltantes.
- **C4. Subclasses:** só 1 CNPJ do universo tem subclasses (`11952873000110`, 2). Aplica-se a regra da seção 3 do protocolo.

## Continuidade medida (11/2015 a 10/2025)

| Grupo | Fundos | Com os 120 meses | Observação |
|---|---|---|---|
| Passivos de varejo | 9 | 9 | |
| ETF de Ibovespa | 1 | 0 | ver D1 |
| Ações ativos | 72 | 57 | 69 com ≥ 36 meses; 68 com ≥ 60 |
| Multimercado | 371 | 217 | 310 com ≥ 36 meses; 270 com ≥ 60 |

- Os 453 fundos têm informe na janela.
- Das 170 séries que terminam antes de 10/2025, 169 têm cancelamento no cadastro (mediana de 2 dias de diferença). A exceção é o BOVA11.
- Não há onda de quebras na adaptação à RCVM 175: terminaram 7 séries em 2023 e 9 em 2024, contra 11 a 26 por ano de 2016 a 2022. Apenas 12 das 295 que chegam a 2025 param antes de outubro.
- Os layouts A, B e C se confirmam nos 64 arquivos. O manifesto com SHA-256 de cada arquivo está em `dados/manifesto_informes.tsv`.
