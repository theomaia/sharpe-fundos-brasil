# Protocolo de seleção de fundos (v1, congelado)

Congelado em 2026-11-02, antes de abrir qualquer retorno. Qualquer mudança posterior deve ser registrada como desvio, com data e motivo.
Até o congelamento foram lidos apenas cadastro, PL e cotistas de 10/2015, e o cabeçalho e 2 linhas de exemplo de alguns arquivos. **Nenhum retorno foi calculado.**

O universo declarado está em [universo_t0.csv](universo_t0.csv).

## 1. Formato dos dados (verificado em 2026-11-02)

### Informe diário: `dados.cvm.gov.br/dados/FI/DOC/INF_DIARIO/`

A frequência é **diária**. O retorno mensal precisa ser calculado a partir de `VL_QUOTA`.

| Período | Onde está | Empacotamento |
|---|---|---|
| 2000 a 2020 | `DADOS/HIST/inf_diario_fi_AAAA.zip` | Anual; 2005, 2017 e 2020 têm um CSV por mês; 2000, 2002 e 2004 têm um CSV único |
| 01/2021 a 10/2026 | `DADOS/inf_diario_fi_AAAAMM.zip` | Mensal, um CSV |

Todos os CSV usam `;` como separador, ponto decimal e datas `AAAA-MM-DD`. O cadastro está em latin-1.

| Layout | Vigência observada | Colunas |
|---|---|---|
| A | até 03/2020 | `CNPJ_FUNDO;DT_COMPTC;VL_TOTAL;VL_QUOTA;VL_PATRIM_LIQ;CAPTC_DIA;RESG_DIA;NR_COTST` |
| B | 04/2020 a 11/2023 | `TP_FUNDO` + layout A |
| C | a partir de 12/2023 | `TP_FUNDO_CLASSE;CNPJ_FUNDO_CLASSE;ID_SUBCLASSE;DT_COMPTC;VL_TOTAL;VL_QUOTA;VL_PATRIM_LIQ;CAPTC_DIA;RESG_DIA;NR_COTST` |

Os limites de cada layout foram vistos nos arquivos 202003/202004 e 202311/202312. Os meses entre eles não foram abertos um a um.

### Armadilhas confirmadas

- **Início efetivo:** o CSV de 2000 está praticamente vazio (zip de 3,5 KB, como o de 2001). 2002 tem 11,6 MB descomprimidos e 2004 tem 105,9 MB. A série só é utilizável por volta de 2004-2005.
- **Formato do CNPJ:** o informe usa `00.017.024/0001-53`, e o `registro_*.csv` usa `00332266000131`. É preciso normalizar antes de qualquer join.
- **CNPJ repetido:** `cad_fi.csv` e `registro_fundo.csv` repetem CNPJ (por exemplo, o mesmo CNPJ como `FITVM` cancelado e `FI` ativo). Um join direto duplica fundos: as contagens iniciais deste protocolo saíram infladas por isso (298 em vez de 231 em Ações, antes do E4).
- **Tipo de fundo:** o valor muda com o tempo (`FI` em 2025, `CLASSES - FIF` em 2026). Não serve como filtro.
- **Subclasses:** no layout C, a chave passa a ser (CNPJ, `ID_SUBCLASSE`, data).
- **Revisões:** os meses M e M-1 são atualizados diariamente, e M-2 a M-11 semanalmente, com reapresentações. Arquivos mais antigos ficam estáticos.
- **Sem benchmark:** o informe não traz indexador nem classe. Isso vem do cadastro.
- **Erros no histórico:** `hist_rentab` tem data de início em 2201. Em t0, 7 CNPJ têm mais de uma classe vigente e 5 têm mais de um indexador. Regra: vale a linha com `DT_INI` mais recente.
- **Regex com acento:** no pandas usado aqui, `\bÍNDICE\b` em `.str.contains` não encontrou nomes com acento, e o `re` do Python encontrou. Usar `re`.

### Cadastro: `dados.cvm.gov.br/dados/FI/CAD/DADOS/`

| Arquivo | Conteúdo | Uso aqui |
|---|---|---|
| `cad_fi_hist.zip` (18 MB) | Histórico por atributo com datas de início e fim: `classe`, `condom`, `exclusivo`, `fic`, `gestor`, `publico_alvo`, `rentab`, `sit`, taxas etc. Cobre a partir de 11/2004 | **Principal**: reconstrói o cadastro em t0 |
| `cad_fi.csv` (17 MB) | Foto atual, somente fundos **não adaptados** à RCVM 175; inclui `CLASSE_ANBIMA` | Rótulo de passivo |
| `registro_fundo_classe.zip` (7 MB) | Foto atual de fundo, classe e subclasse, adaptados ou não | Rótulo de passivo dos adaptados; ETFs |

**Domínios confirmados em t0:** `CLASSE` = `Ações`, `Multimercado` etc.; `CONDOM` = `Aberto`/`Fechado`; `FUNDO_EXCLUSIVO` = `S`/`N`; `RENTAB_FUNDO` = `Ibovespa`, `IBrX`, `DI de um dia`, `OUTROS` etc.; rótulo ANBIMA = `Ações Indexados`, `Ações Índice Ativo`, `Fundos de Mono Ação` etc.

**Limite do cadastro histórico:** não há histórico da classificação ANBIMA. O rótulo é uma foto de hoje, e quase sempre falta em fundos cancelados.

**ETFs ficam de fora do `cad_fi_hist`,** mas estão no informe diário. Exemplo: `10.406.511/0001-61`, ISHARES IBOVESPA, tipo `FIIM` no `registro_fundo`.

### Taxa livre de risco

CDI mensal do BCB, série SGS 4391, em % ao mês (testada: 10/2015 = 1,11; 10/2026 = 1,02).

### Arquivos usados nas contagens (baixados em 2026-11-02)

| Arquivo | SHA-256 |
|---|---|
| `cad_fi.csv` | `09f869269f6e5cbe0e54b7b0f47dde77653cf0814175658da6f3f385700e2ed0` |
| `cad_fi_hist.zip` | `2665ec64638056ebaab7662d74d21d7a475992b5fcd2a350a99ac1fbc8f24d91` |
| `registro_fundo_classe.zip` | `64508cfc1e01a1ba5ce60c0bc57dc4983876e470ce0c3590686119958d0953a2` |

## 2. Janela

- **Data de formação t0:** 30/10/2015.
- **Janela de teste:** 11/2015 a 10/2025, T = 120 meses, igual ao paper original.
- **Motivo do fim em 10/2025:** é o último mês já fora do ciclo de reapresentação (M-12 em relação a 10/2026).
- **Efeito útil:** a seleção usa apenas `cad_fi_hist` e o informe de 10/2015, sem passar pela transição da RCVM 175.

## 3. Retornos

- r(t) = ln(Q_t / Q_{t-1}) − ln(1 + CDI_t/100), com Q_t = última cota do mês. Log em excesso, como em Ledoit-Wolf.
- A última cota precisa estar nos 5 últimos dias úteis do mês. Caso contrário, o mês fica faltante.
- O retorno é líquido de taxas e bruto de IR (o come-cotas reduz o número de cotas, não o valor da cota).
- Variações diárias acima de 50% em módulo são listadas e tratadas **antes** de calcular qualquer Sharpe. Só se corrige o que for evento de cota documentado.
- Subclasses: quando um CNPJ tiver mais de uma, usa-se a de maior PL na primeira data em que aparece.

## 4. Elegibilidade em t0

Todos os critérios usam apenas informação disponível até t0.

| # | Critério | Fonte | Objetivo |
|---|---|---|---|
| E1 | `CLASSE` vigente = `Ações` ou `Multimercado` | `hist_classe` | Definir a categoria sem look-ahead |
| E2 | `CONDOM` = `Aberto` | `hist_condom` | Fundo investível |
| E3 | Não exclusivo | `hist_exclusivo` | Remover carteiras de um único cotista |
| E4 | `EM FUNCIONAMENTO NORMAL` iniciado até 30/10/2013 | `hist_sit` | Viés de incubação, sem precisar abrir cotas |
| E5 | ≥ 100 cotistas em 30/10/2015 | Informe | Remover fundos master e fechados na prática |
| E6 | PL ≥ R$ 10 mi em 30/10/2015 | Informe | Remover fundos residuais |

**Não se exige sobrevivência até o fim da janela.** Exigir T completo reintroduziria o viés de sobrevivência que os dados da CVM permitem evitar.

### Contagens

| Etapa | Ações | Multimercado |
|---|---|---|
| Passam em E1-E6 | 218 | **371** |
| Com indexador Ibovespa, sem ação única | 81 (9 passivos + 72 ativos) | n/a |
| Gestores distintos | 31 (ativos) | 108 |
| FICs | 46 (ativos) | 282 |

Atrito cadastral, usado só para justificar a regra acima e não para selecionar:

- Dos 72 ativos, 16 foram cancelados até 10/2025 (22%). Dos 9 passivos, nenhum.
- Dos 371 multimercados, 154 foram cancelados até 10/2025 (42%).
- Cancelamento inclui incorporação; não significa necessariamente fracasso do fundo.

## 5. Aplicações

### Aplicação 1: ações, ativo × passivo

- **Passivo:** indexador Ibovespa em t0 e (rótulo ANBIMA `Ações Indexados` ou nome com `INDEX`, `PASSIV` ou `ÍNDICE`), sem `ATIVO` no nome nem rótulo `Ações Índice Ativo`.
- **ETF passivo:** tipo `FIIM`, replica o Ibovespa, em t0 passa em E4-E6 (usando `Data_Registro`). Em t0 há um que atende: ISHARES IBOVESPA.
- **Ativo:** indexador Ibovespa em t0, não passivo, sem ação única (rótulo `Fundos de Mono Ação` ou nome com `PETROBRAS`, `VALE`, `BANCO DO BRASIL`, `ITAÚSA`). 8 fundos excluídos por esse último filtro.

**Passivo principal (decidido em 2026-11-02): ISHARES IBOVESPA (BOVA11),** `10.406.511/0001-61`, R$ 1.806 mi em t0, 5.953 cotistas. É o maior passivo pela mesma regra de PL, com mandato sem ambiguidade.

**Passivo alternativo (robustez):** ITAÚ PERSONNALITÉ AÇÕES INDEX IBOVESPA (`67.976.449/0001-60`), R$ 76,6 mi, 3.286 cotistas, maior passivo de varejo.

Nos passivos de varejo, a taxa de administração cadastrada vai de 2,0% a 4,0% (mediana de 2,5%, com uma exceção de 0,0% a conferir), contra mediana de 3,0% nos ativos. Com o passivo de varejo, ativo × passivo viraria em boa parte uma comparação entre fundos de custo parecido.

As taxas só servem para descrever os fundos e não entram em nenhum teste, porque `VL_QUOTA` já é líquida de taxas. O dicionário da CVM define `TAXA_ADM` apenas como "Taxa de administração", sem dizer se é máxima ou efetiva. Para o BOVA11, que não consta no histórico de taxas, as fontes secundárias divergem (0,10% e 0,54% ao ano), e nenhum regulamento foi lido; no paper, citar taxa só depois de ler o regulamento de cada período.

- **1a, par:** BOVA11 × ativo de maior PL com **≥ 1.000 cotistas**. Resultado: GERAÇÃO FUTURO FIA PROGRAMADO IBOVESPA ATIVO (`07.972.299/0001-95`, R$ 503 mi), cancelado em 09/2018, o que aciona a regra de encerramento.
- **1b, agregado (teste principal):** carteira igualmente ponderada dos 72 ativos, rebalanceada mensalmente, contra o BOVA11. Fundo encerrado sai, e seu peso é redistribuído.

**Por que o piso de 1.000 cotistas nos pares:** sem ele, o maior ativo é o TNA FIC FIA (118 cotistas, R$ 576 mi, hoje rotulado como investimento no exterior), que parece veículo fechado e não fundo de varejo comparável. Todos os passivos têm mais de 1.000 cotistas. **Esse piso foi definido depois de ver nomes e a data de cancelamento**, embora nenhum retorno, e deve ser declarado assim. O par sem piso entra como robustez.

### Aplicação 2: multimercado

Os dois maiores PL em t0, de gestores distintos, com ≥ 1.000 cotistas (77 fundos):

- SAFRA GALILEO FI MULTIMERCADO (`10.347.249/0001-21`, R$ 9.997 mi)
- SPX NIMITZ FEEDER FIQ FI MULTIMERCADO (`12.831.360/0001-14`, R$ 2.795 mi)

Sem o piso, o segundo seria CSHG ALL VERDE AM PRISMA (848 cotistas, R$ 3.117 mi).

### Encerramento de fundo selecionado

A posição migra para o maior fundo elegível do mesmo grupo na data do encerramento, aplicando os mesmos filtros naquela data. A série sintética é reportada como tal. Não se troca o fundo por outro que sobreviveu até o fim.

### Teste (parâmetros do paper)

- H0: igualdade de Sharpe; bicaudal; α = 5%.
- Reportar os 5 métodos, como na Tabela 3 original: JKM, HAC, HAC prewhitened, Boot-IID e Boot-TS.
- Boot-TS: bloco circular, b escolhido em {1, 2, 4, 6, 8, 10} pelo Algoritmo 3.1, K = 5000, M = 4999.
- Modelo da calibração: VAR(1) com stationary bootstrap dos resíduos, bloco médio 5.

## 6. Deflated Sharpe

- **Grupo:** os **M = 371** multimercados elegíveis em t0, listados em `universo_t0.csv`, inclusive os que encerraram depois.
- **Sharpe:** mensal, **não anualizado**.
- **V[SR] e vencedor:** fundos com ≥ 36 meses na janela. Reportar quantos ficaram de fora.
- **N:** o paper define N como número de tentativas **independentes**.
  - Principal: N̂ = ρ̂ + (1 − ρ̂)·M (Apêndice A.3, Eq. 9), com ρ̂ = correlação média.
  - Reportar também com N = M, como limite conservador.
- **Critério:** DSR > 0,95.

Limitações a declarar:

- A variância entre fundos mistura ruído e habilidade real, o que tende a elevar SR0.
- A H0 é Sharpe verdadeiro igual a zero, e a média dos Sharpe entre fundos pode não ser zero. Reportar sensibilidade somando E[SR] da Eq. 1.
- Clones inflam M: são 282 FICs em 371 fundos e 108 gestores. Por isso a correção por N̂ é a principal.
- A classe CVM `Multimercado` inclui fundos de crédito: 41 têm crédito privado no nome, e alguns hoje têm rótulo ANBIMA de renda fixa.

## 7. Robustez (declarada antes de ver resultado)

1. Subjanelas de 60 meses, com os mesmos fundos.
2. Janela anterior: t0 = 10/2005, 11/2005 a 10/2015. E4 limitado a 11 meses, porque o histórico começa em 11/2004.
3. Datas de formação alternativas: cada outubro de 2013 a 2017, para ver se o par muda.
4. DSR com mínimo de 60 e 120 meses; DSR sem os 41 fundos de crédito privado no nome (M = 330).
5. Carteira 1b ponderada por PL; ponderada igualmente **por gestor**, porque os 72 ativos vêm de 31 gestores; e sem exigir indexador Ibovespa (182 ativos).
6. Passivo alternativo: Itaú Personnalité Index Ibovespa no lugar do BOVA11.
7. Pares sem o piso de 1.000 cotistas.

## 8. Limitações do desenho

- **O paper original não compara ativo × passivo.** Os pares eram FFIDX × FDEGX (dois fundos ativos) e Coast Enhanced Income × JMG Capital Partners. Os autores dizem apenas "selected funds", sem critério, como ilustração do método.
- **Um par não responde a uma pergunta sobre a população.** O par 1a mostra o teste funcionando; quem responde "ativo supera passivo?" é o 1b.
- **A Aplicação 2 não testa ativo × passivo,** porque não há multimercado passivo. Ela testa se a inferência muda entre os métodos.
- **Sharpe negativo:** com Sharpe negativo, o maior risco melhora o índice, e o ordenamento fica enganoso. Se a média em excesso for negativa, isso será reportado junto com o resultado.
- **Rótulo de passivo:** é uma foto de hoje, e nenhum dos 9 passivos foi cancelado até 10/2025. Passivos mortos com nomes sem `INDEX`/`ÍNDICE` podem estar entre os ativos.
- **BOVA11 é negociado em bolsa:** o retorno pela cota ignora corretagem, spread e a diferença entre preço de mercado e cota. O investidor real tem esses custos, e o cotista de um fundo ativo não, o que favorece um pouco o passivo.
- **Falsos positivos:** BB AÇÕES ÍNDICE DE SUSTENTABILIDADE está no grupo passivo, com indexador Ibovespa no cadastro, embora o nome aponte outro índice. Não afeta os testes, que usam só o maior passivo.

## 9. A verificar

- [x] Domínios exatos de classe, indexador e rótulo ANBIMA.
- [x] Contagens em t0: existem 9 passivos de varejo e 1 ETF de Ibovespa.
- [x] Cobertura em t0: de 13.057 CNPJ com informe na última semana de 10/2015, 119 estão sem classe e 1.750 sem indexador (ETFs ajudam a explicar esses casos, sem contagem exata).
- [x] ETFs aparecem no informe de 10/2015.
- [x] Passivo principal: BOVA11 (seção 5).

**Depois de congelar (não alteram a seleção):**

- [ ] Tornar a data verificável fora da máquina local (GitHub ou OSF). O commit local sozinho não prova a data.
- [ ] Continuidade do CNPJ: todos os elegíveis nunca cancelados têm o mesmo CNPJ no `registro_classe`, mas a continuidade das **cotas** só pode ser medida ao abrir os informes. Inclui a série completa do BOVA11.
- [ ] Taxas de BOVA11 e dos fundos citados, nos regulamentos, para a parte descritiva do paper.
- [ ] Reproduzir a Tabela 2 de Ledoit-Wolf (FFIDX × FDEGX) como teste da implementação, comparando com o código público de Wolf.

## Fontes

- Ledoit & Wolf (2008), *Journal of Empirical Finance* 15(5), 850-859. Texto lido: seções 3 e 6, Tabelas 2 e 3.
- Bailey & López de Prado (2014), *Journal of Portfolio Management* 40(5), 94-107. Texto lido: Eq. 1, Eq. 2, Apêndice A.3.
- Dicionários da CVM: `meta_inf_diario_fi.txt`, `meta_cad_fi.txt`, `meta_registro_fundo_classe.zip`.
