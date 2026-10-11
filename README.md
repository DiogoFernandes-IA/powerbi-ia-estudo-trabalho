# IA no estudo e no trabalho

**Autor:** Diogo Fernandes · **Ferramentas:** Power BI Desktop, Power Query (M) e DAX

## Objetivo

Construir um relatório interativo que descreva o uso de ferramentas de inteligência artificial entre estudantes e profissionais, comparando frequência de ferramentas, horas de uso diário, tempo economizado por semana e distribuição dos registros por país. Projeto de portfólio desenvolvido no contexto da formação em Dados e Analytics.

O relatório responde: quais ferramentas são mais frequentes nesta base? Como os perfis diferem em uso diário e economia de tempo? Como os usuários da amostra se distribuem entre países?

## Fonte e escopo

- Dataset: [AI Usage and Impact on Students and Professionals — Kaggle](https://www.kaggle.com/datasets/eishatuzzuhra/ai-usage-and-impact-on-students-and-professionals).
- Licença informada na fonte pelo autor do projeto: [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).
- Base original: 300 registros e 19 colunas. Após remoção de 12 duplicatas exatas: 288 registros.
- Granularidade utilizada: um registro por ID de usuário na base tratada.
- Uma tabela auxiliar de país e continente foi criada manualmente para enriquecer a análise geográfica. Ela não faz parte do dataset original.

Não foi validada nesta documentação a metodologia de coleta, a representatividade ou a natureza real ou sintética dos dados. Portanto, os resultados descrevem exclusivamente esta base e não estimam a adoção de IA na população.

## Preparação dos dados

| Etapa | Decisão e finalidade |
|---|---|
| Preservação da origem | Manter Base_Original e separar Base_Tratada; desabilitar a carga da consulta original. |
| Duplicatas | Remover 12 linhas integralmente duplicadas, preservando 288 registros. |
| Tradução | Traduzir cabeçalhos e padronizar categorias para leitura em português. |
| Texto | Limpar caracteres e espaços nas categorias; corrigir variações de países, perfis e ferramentas. |
| Idade | Remover sufixos textuais e converter para inteiro; preservar ausências. |
| Medidas em horas | Remover sufixos de horas e converter uso diário e tempo economizado para decimal, interpretando o ponto decimal da origem. |
| Custo | Remover símbolos textuais e converter para número. A unidade monetária não foi validada; não utilizar este campo para conclusões financeiras. |
| Ausências categóricas | Substituir null e texto vazio em País e Ferramenta de IA por Não informado. |
| Ausências numéricas | Preservar null; não substituir por zero, evitando distorção nas médias. |
| Novas colunas | Criar Faixa Etária e Categoria de custo. |
| Enriquecimento | Mesclar Base_Tratada com AuxPaisContinente pela coluna País, usando junção externa à esquerda; expandir somente Continente. |
| Validação da mesclagem | Confirmar correspondência de 288 registros e manutenção de 288 linhas após expansão. |

Campos retirados da base tratada: Atividades Realizadas, Profissão ou Curso, Renda Mensal, Trabalho ou Estudo por Dia, Pontuação de Produtividade, Avaliação de Precisão, Pontuação de Satisfação e Recomendaria a IA. O recorte concentra o relatório em uso, economia de tempo, ferramentas e geografia; os campos originais permanecem na consulta de origem.

A tabela AuxPaisContinente contém uma única linha para cada país da base, incluindo Não informado. Sua carga foi desabilitada, mantendo seu uso no Power Query. A classificação separa América do Norte e América do Sul.

## Modelo de dados

Base_Tratada ocupa o centro de um modelo em estrela simplificado, com dimensões derivadas dos valores únicos da base.

| Dimensão | Chave de relacionamento | Cardinalidade e filtro |
|---|---|---|
| DimPais | País | 1 para muitos; dimensão → Base_Tratada |
| DimPerfil | Perfil do Usuário | 1 para muitos; dimensão → Base_Tratada |
| DimFerramenta | Ferramenta de IA | 1 para muitos; dimensão → Base_Tratada |

Os três relacionamentos são ativos, usam chaves textuais e direção única. As dimensões permitem organizar filtros e agrupamentos. A junção no Power Query acrescenta uma coluna durante o tratamento; os relacionamentos do modelo propagam filtros durante a análise. São operações distintas.

## Medidas DAX

```dax
Total de Usuários =
DISTINCTCOUNT('Base_Tratada'[ID do Usuário])

Uso Médio Diário de IA =
AVERAGE('Base_Tratada'[Uso de IA por Dia (h)])

Tempo Médio Economizado por Semana =
AVERAGE('Base_Tratada'[Tempo Economizado por Semana (h)])

Participação dos Usuários por País =
DIVIDE(
    [Total de Usuários],
    CALCULATE(
        [Total de Usuários],
        REMOVEFILTERS('DimPais')
    )
)
```

As médias desconsideram valores vazios. A participação por país utiliza o total no contexto dos demais filtros, removendo filtros de DimPais. Para esse comportamento, os campos geográficos do visual e os filtros de país devem vir de DimPais. O total das médias é recalculado sobre os registros válidos, não pela média simples das médias dos países. A participação é formatada como porcentagem com uma casa decimal.

## Páginas e interações

**Visão geral:** cartões com total de usuários, uso médio diário e tempo médio economizado; ranking de ferramentas; comparação dos dois perfis; mapa da distribuição dos usuários por país. O perfil é filtrado por segmentação. Não informado é excluído apenas do mapa, preservando os registros nos demais visuais.

**Análise por país:** tabela com usuários, participação, uso médio diário e tempo médio economizado. Inclui segmentação de perfil, filtro de página por ferramenta, explicação metodológica e resumo de resultados. As segmentações das páginas não são sincronizadas. As conclusões em caixa de texto são estáticas e identificadas como resumo da amostra completa.

## Resultados e interpretação

| Indicador, sem filtros | Resultado |
|---|---:|
| Usuários distintos | 288 |
| Uso médio diário de IA | 2,94 h/dia |
| Tempo médio economizado | 5,81 h/semana |
| Usuários associados a ChatGPT | 119 (41,3%) |
| Usuários da Índia | 43 (14,9%) |
| País não informado | 10 (3,5%) |
| Ferramenta não informada | 10 (3,5%) |

**Concentração por ferramenta.** ChatGPT é a categoria mais frequente, com 119 registros, seguido por Google Gemini (39), Microsoft Copilot (33) e Claude (31). A participação de 41,3% usa os 288 usuários, incluindo ferramenta não informada. Trata-se de frequência nesta base; não é participação de mercado nem comprovação de superioridade técnica.

**Diferença entre perfis.** Os gráficos mostram aproximadamente 6,0 h/semana de economia média entre estudantes e 5,6 entre profissionais, com uso diário próximo de 3,0 e 2,9 horas, respectivamente. A diferença descritiva sugere investigar os tipos de tarefa e os contextos de uso em um estudo posterior. Não demonstra que pertencer a um perfil causa maior economia de tempo, nem que a diferença é estatisticamente significativa.

**Distribuição geográfica.** A Índia reúne o maior número de usuários da amostra (43), seguida por Estados Unidos (34) e Canadá (31). Esses números refletem a composição da base, sem ajuste pela população de cada país.

**Tamanho dos grupos.** O Quênia apresenta as maiores médias geográficas exibidas: 5,03 h/dia e 9,15 h/semana de economia, mas tem apenas seis usuários. O resultado exige cautela: grupos pequenos podem ser muito sensíveis a poucos registros. Não sustenta um ranking nacional de produtividade.

**Uso e benefício são medidas diferentes.** A Nigéria apresenta uso médio de 3,97 h/dia e economia de 5,44 h/semana; o Canadá, 3,64 h/dia e 6,49 h/semana. O contraste mostra por que as duas medidas precisam ser examinadas separadamente. Essa comparação entre médias de países não mede a correlação entre uso e economia no nível do usuário.

## Limitações e próximos estudos

- Não há evidência suficiente para generalizar os resultados aos países ou às categorias profissionais.
- As horas são valores registrados no dataset; a documentação de coleta precisa ser verificada antes de apresentá-las como benefícios medidos ou respostas de pesquisa.
- Ausências numéricas fazem com que o número de observações válidas varie entre medidas e grupos. Uma evolução útil é exibir essas contagens em dicas de ferramenta.
- O painel não comprova causalidade, retorno financeiro ou crescimento de produtividade.
- Não há análise temporal; o relatório não deve sugerir evolução da adoção ao longo do tempo.
- Como extensão, comparar medianas e distribuições, verificar valores extremos e investigar a relação entre uso e economia com os registros individuais.

## Validação realizada durante a construção

Foram conferidos os totais gerais (288; 2,94; 5,81), as interações da segmentação de perfil, o filtro por ferramenta na página de países, a participação total de 100% e a preservação da contagem após a mesclagem. As duas páginas foram revisadas em PDF; ajustes posteriores de texto e layout foram realizados no Desktop pelo autor. A última exportação com todos esses ajustes ainda deve ser conferida antes da publicação.

## Entrega do portfólio

Organizar juntos o arquivo IA_Estudo_Trabalho.pbix, a última exportação IA_Estudo_Trabalho.pdf, o print Modelo_Dados_IA.png e esta documentação. O PBIX é o relatório interativo; o PDF facilita a leitura sem Power BI; o print mostra o modelo; este arquivo descreve decisões e limitações.

Para reproduzir, baixar o CSV da fonte, atualizar o caminho do arquivo na consulta de origem no Power Query e atualizar o relatório. Os nomes das colunas e o formato da fonte devem corresponder à versão utilizada. Registrar a data de download e o nome exato do CSV no repositório ao organizar a entrega.

**Competências demonstradas:** preparação de dados, preservação da origem, padronização de categorias, tratamento de ausências, tipagem com localidade, criação de colunas, junção externa à esquerda, modelagem dimensional, medidas DAX, contexto de filtro, visualização geográfica e comunicação dos limites da análise.
