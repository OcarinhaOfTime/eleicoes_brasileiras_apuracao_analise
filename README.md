# Evolução da apuração das eleições presidenciais brasileiras

Um pequeno projeto em **Python, Pandas e Plotly** para visualizar a evolução das proporções de votos nas eleições presidenciais de **2018, 2022 e 2026**, nos dois turnos.

## Objetivo

A proposta é **facilitar o acesso aos dados eleitorais e a experimentos reproduzíveis**, contribuindo para uma discussão pública mais transparente e fundamentada em evidências, especialmente diante da circulação de desinformação sobre eleições.

O notebook reúne séries históricas, reconstrói a evolução de 2018 a partir de registros por seção e apresenta gráficos interativos. A finalidade é permitir que qualquer pessoa examine como os resultados acumulados evoluíram e confronte afirmações sobre seus supostos padrões.


## Dados utilizados

As fontes e os procedimentos **não são idênticos entre os anos**. Essa diferença é central para interpretar os gráficos:

| Eleição | Fonte efetivamente utilizada | Como a curva é obtida | Principal limitação |
| --- | --- | --- | --- |
| **2018 · 1º e 2º turnos** | Arquivos oficiais de votação por seção e de detalhe da apuração do **TSE**; reconstrução inspirada no **Números da Eleição** | Os votos finais de cada seção são associados ao horário de sua primeira totalização (`DT_PRIM_TOT_PARCIAL_HOR_TSE`), agrupados por minuto e acumulados | **Reconstrução cronológica**, não gravação das parciais publicadas; não reproduz eventuais correções intermediárias |
| **2022 · 1º e 2º turnos** | Arquivos **Histórico da Totalização — Presidente**, publicados pelo **TSE** | Leitura direta das atualizações acumuladas oficiais | É a série histórica mais detalhada usada aqui, mas cada registro representa uma atualização, não um voto individual |
| **2026 · 1º turno** | Série de parciais do TSE preservada pelo projeto **gfvdata** | Leitura dos percentuais acumulados publicados nos registros arquivados | Série de **capturas selecionadas**, com menos observações; não contém os votos absolutos por seção |
| **2026 · 2º turno** | Parciais do TSE preservadas pelo projeto **Contagem Direta**, disponibilizadas no repositório de **João Gallo** | Leitura de capturas com seções, votos acumulados e horários | As **367 observações válidas** presentes no arquivo utilizado não equivalem a todas as atualizações da totalização |

Em 2022, o segundo turno possui aproximadamente **5,3 mil atualizações** no histórico oficial; em 2026, o arquivo do segundo turno oferece **367 registros válidos**.

No **primeiro turno**, são exibidos apenas os dois candidatos selecionados no notebook; seus percentuais **não precisam somar 100%**, pois existem outros candidatos. No segundo turno, as duas proporções dos votos válidos são complementares.

### Links das fontes

**TSE — dados oficiais**

- [Resultados de 2018 — Portal de Dados Abertos](https://dadosabertos.tse.jus.br/dataset/resultados-2018)
- [2018 — votação presidencial por seção (ZIP)](https://cdn.tse.jus.br/estatistica/sead/odsele/votacao_secao/votacao_secao_2018_BR.zip)
- [2018 — detalhe da votação por seção (ZIP)](https://cdn.tse.jus.br/estatistica/sead/odsele/detalhe_votacao_secao/detalhe_votacao_secao_2018.zip)
- [Resultados de 2022 — Portal de Dados Abertos](https://dadosabertos.tse.jus.br/dataset/resultados-2022)
- [2022 — histórico de totalização presidencial, 1º turno (ZIP)](https://cdn.tse.jus.br/estatistica/sead/eleicoes/eleicoes2022/Historico_Totalizacao_Presidente_BR_1T_2022.zip)
- [2022 — histórico de totalização presidencial, 2º turno (ZIP)](https://cdn.tse.jus.br/estatistica/sead/eleicoes/eleicoes2022/Historico_Totalizacao_Presidente_BR_2T_2022.zip)
- [Resultados de 2026 — Portal de Dados Abertos](https://dadosabertos.tse.jus.br/dataset/resultados-2026)
- [TSE — como funciona a totalização dos resultados](https://www.tse.jus.br/eleicoes/historia/processo-eleitoral-brasileiro/totalizacao)

**Projetos independentes e metodologia**

- [Números da Eleição — fontes e reconstrução de 2018](https://numerosdaeleicao.com.br/fontes) — **referência metodológica**; o notebook **não baixa** séries prontas desse projeto.
- [gfvdata — repositório das eleições de 2026](https://github.com/gfvdata-web/eleicoes-2026) e [documentação da apuração](https://github.com/gfvdata-web/eleicoes-2026/blob/main/documentacao/apuracao.md) — fonte das capturas do primeiro turno.
- [gfvdata — arquivo efetivamente utilizado (JSON)](https://raw.githubusercontent.com/gfvdata-web/eleicoes-2026/main/docs/data/apuracao/evolucao-presidente.json).
- [João Gallo — projeção e acervo da apuração de 2026](https://github.com/joaogallo/projecao-apuracao-eleicoes) — repositório que incorpora a série arquivada pelo **Contagem Direta** para o segundo turno.
- [João Gallo — arquivo efetivamente utilizado (JSON)](https://raw.githubusercontent.com/joaogallo/projecao-apuracao-eleicoes/main/resultados/dashboard.json).

## Como executar

É necessário **Python 3** com as bibliotecas `numpy`, `pandas`, `requests`, `plotly` e um ambiente capaz de abrir notebooks Jupyter (por exemplo, **VS Code com a extensão Jupyter**).

```bash
python -m pip install numpy pandas requests plotly notebook
jupyter notebook eleicoes.ipynb
```

Execute primeiro as células de **imports/configuração** e **funções**. Para carregar e visualizar **uma eleição por vez**, utilize a célula de seleção:

```python
YEAR, ROUND = 2022, 2
X_AXIS = "progress"

df = load_counting(YEAR, ROUND)
fig = plot_counting(df, YEAR, ROUND, x_axis=X_AXIS)
```

Use `YEAR = 2018`, `2022` ou `2026` e `ROUND = 1` ou `2`.

**Atenção:** a terceira célula do notebook é uma rotina opcional de **carregamento em lote dos seis cenários**. **Pule essa célula** se quiser examinar apenas uma eleição. A primeira execução de 2018 baixa arquivos grandes e processa os dados em blocos; os resultados reconstruídos são guardados em cache na pasta `data/`. Os demais downloads também são reutilizados quando os arquivos locais já existem.

Se o gráfico não aparecer fora do VS Code, altere `fig.show(renderer="vscode")` para `fig.show()` na função `plot_counting`.

## Funções implementadas

| Função | O que faz |
| --- | --- |
| `download(key)` | Obtém os arquivos das URLs configuradas e reaproveita downloads locais. |
| `load_2018(round_)` | Reconstrói a curva de 2018 a partir dos votos finais e horários de primeira totalização; agrupa por minuto e utiliza cache. |
| `load_counting(year, round_)` | Lê o histórico adequado a cada eleição e padroniza o retorno (`progress`, `share_a`, `share_b` e, quando disponíveis, `sections`, `votes_a`, `votes_b`, `validos`, `timestamp`). |
| `plot_counting(df, year, round_, ...)` | Produz o gráfico interativo de proporções acumuladas, com títulos, legendas e eixos em português. |

As proporções são calculadas em relação ao **total acumulado de votos válidos**. Quando disponíveis os totais absolutos, a definição é:

\[
\text{proporção do candidato} = \frac{\text{votos acumulados do candidato}}{\text{votos válidos acumulados}}
\]

O notebook inclui algumas verificações de consistência dos resultados finais, mas isso **não equivale a uma validação independente de todas as observações**.

## Eixos e opções de visualização

É importante distinguir **forma de visualização** de **método de reconstrução**:

| Opção | Significado | Quando usar |
| --- | --- | --- |
| `x_axis="progress"` | **Percentual de seções totalizadas**, de 0 a 100. | Facilita a leitura do avanço de cada eleição em uma escala comum. É o padrão. |
| `x_axis="sections"` | **Número acumulado de seções**. | Permite observar o avanço em números absolutos. Disponível em **2018, 2022 e 2026/2º turno**, mas **não** na série utilizada de **2026/1º turno**. |
| `step=None` | Mantém **todas as observações disponíveis** na fonte carregada. | Melhor opção para examinar os detalhes preservados de uma eleição. |
| `step=0.25` | Exibe o último registro disponível até cada intervalo de **0,25 ponto percentual** de progresso, sem interpolar votos. | Útil para reduzir a densidade visual; **não recupera** atualizações ausentes. |
| `x_range=(10, 50)` | Mostra apenas o intervalo escolhido do eixo horizontal. | Inspecionar uma etapa específica da totalização. |
| `y_range=(0.45, 0.55)` | Limita o eixo vertical a proporções entre **45% e 55%**. | Ampliar pequenas diferenças entre candidatos. |

Por exemplo:

```python
fig = plot_counting(
    df, YEAR, ROUND,
    x_axis="progress",
    step=None,
    x_range=(10, 50),
    y_range=(0.45, 0.55),
)
```

### E o modo `geographic` ou `random`?

Esses modos apareceram em **experimentos anteriores**, mas **não estão implementados neste notebook**. Eles representam outra abordagem: **reordenar artificialmente as seções** (por localização geográfica ou por sorteio) e recalcular os resultados acumulados. Isso cria **simulações contrafactuais**, não a ordem efetiva da apuração. Podem ser úteis para investigar como a ordenação influencia a forma das curvas, mas seus resultados **não devem ser apresentados como históricos reais**.

Nesta versão, `progress` e `sections` são **eixos**, não modos de simulação. A reconstrução cronológica de **2018** segue horários registrados; **2022** usa o histórico oficial; **2026** usa parciais arquivadas por terceiros.

## Limitações e interpretação

- **Granularidade desigual:** 2018 agrupa seções pelo minuto de primeira totalização; 2022 utiliza atualizações oficiais; 2026 utiliza capturas arquivadas de frequência menor. Comparações de **micro-oscilações**, frequência de inversões e supostas anomalias precisam considerar essa diferença.
- **Cronologia de 2018 reconstruída:** os votos são finais, mas sua entrada é aproximada pelo horário de primeira totalização. A curva não representa necessariamente cada estado intermediário publicado ao eleitor.
- **Cobertura de 2026:** arquivos de projetos independentes preservam parciais do TSE, mas podem omitir atualizações intermediárias. Seu conteúdo pode mudar se os repositórios de origem forem atualizados.
- **Eixo de progresso:** percentual de **seções** totalizadas não equivale necessariamente ao percentual de **votos** totalizados, pois as seções têm tamanhos diferentes.
- **Dados acumulados:** curvas suavizadas, inclinações e cruzamentos de liderança não constituem, isoladamente, testes de fraude. Sua interpretação depende da composição geográfica dos lotes, da logística de transmissão e de hipóteses estatísticas explícitas.
- **Escopo:** este é um instrumento **exploratório de visualização**. Não implementa auditoria criptográfica, detecção automática de anomalias, inferência causal, testes formais de fraude ou estimativas contrafactuais de resultado.

Para pesquisas que dependam de valores e versões exatos, recomenda-se **registrar a data de acesso e o hash dos arquivos baixados**, especialmente os JSONs hospedados em branches `main` de terceiros.

## Reprodutibilidade e colaboração

O notebook utiliza uma pasta local `data/` para downloads e cache. Ao publicar o repositório, é recomendável adicionar `data/` ao `.gitignore`, evitando versionar arquivos volumosos; eles podem ser recuperados pelos links acima.

Contribuições, revisões de código, correções de fontes e propostas de testes estatísticos são bem-vindas. O objetivo é manter uma base acessível, verificável e aberta à crítica — **sem pressupor conclusões políticas ou eleitorais antes da análise dos dados**.

---

**Nota:** este repositório é um projeto independente de exploração de dados públicos. Não possui vínculo institucional com o TSE, com o Números da Eleição ou com os projetos que preservaram as parciais de 2026.
