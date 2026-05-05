# Do que se trata
Base municipal brasileira de climatologia histórica mensal derivada do TerraClimate. O arquivo agrega temperatura mínima, máxima, média, precipitação, evapotranspiração potencial, evapotranspiração real e déficit hídrico para os municípios do Brasil, usando média espacial dos pixels do raster dentro do território municipal.

# Clima Historico Municipal do Brasil a partir do TerraClimate

Este documento acompanha o arquivo `data/clima_historico_municipal.csv`, produzido no projeto **GeoBrasil Inteligente**. A base reúne, em formato tabular leve, médias climáticas mensais históricas para todos os municípios brasileiros, derivadas de dados do TerraClimate e agregadas para o recorte municipal. O objetivo é disponibilizar uma camada climática municipal reutilizável, legível e fácil de integrar com outros indicadores territoriais.

A utilidade desta base está no fato de que informações climáticas históricas com cobertura nacional e já organizadas por município não são triviais de encontrar em formato pronto para uso analítico. É relativamente comum encontrar séries em grade espacial, rasters globais, arquivos NetCDF ou dados por estação meteorológica. O que é menos comum é uma base municipal consolidada, com identificação por `codigo_ibge`, já preparada para cruzamentos com dados socioeconômicos, ambientais, demográficos e territoriais. Este arquivo busca preencher exatamente essa lacuna.

Em vez de exigir que cada pesquisador, gestor, analista ou desenvolvedor refaça o mesmo pipeline geoespacial, o arquivo entrega um produto intermediário já processado: climatologia histórica mensal por município. Isso o torna útil para visualização em mapas, comparação entre territórios, estudos regionais, análise exploratória de sazonalidade, construção de índices compostos e integração com painéis públicos ou aplicações web.

## O que o arquivo contém

O arquivo `clima_historico_municipal.csv` contém um registro por município, identificado por código IBGE, nome e UF. Para cada município, a base informa o período de referência, a fonte climática, o método de processamento e um conjunto de variáveis mensais agregadas historicamente.

As variáveis climáticas presentes são:

- `tmin_*`: temperatura mínima média mensal histórica
- `tmax_*`: temperatura máxima média mensal histórica
- `temp_media_*`: temperatura média mensal histórica, derivada de `tmin` e `tmax`
- `ppt_*`: precipitação mensal histórica
- `pet_*`: evapotranspiração potencial mensal histórica
- `aet_*`: evapotranspiração real mensal histórica
- `def_*`: déficit hídrico mensal histórico

Cada sufixo mensal corresponde a um mês do ano:

- `jan`, `fev`, `mar`, `abr`, `mai`, `jun`, `jul`, `ago`, `set`, `out`, `nov`, `dez`

Assim, por exemplo:

- `temp_media_jan` representa a média histórica de temperatura média para todos os janeiros do período processado
- `ppt_mar` representa a média histórica de precipitação para todos os meses de março do período processado
- `def_set` representa a média histórica de déficit hídrico para todos os meses de setembro do período processado

As unidades utilizadas são:

- temperaturas (`tmin`, `tmax`, `temp_media`): **graus Celsius (°C)**
- precipitação (`ppt`): **milímetros por mês (mm/mês)**
- evapotranspiração potencial (`pet`): **milímetros por mês (mm/mês)**
- evapotranspiração real (`aet`): **milímetros por mês (mm/mês)**
- déficit hídrico (`def`): **milímetros por mês (mm/mês)**

## Fonte dos dados

A fonte climática original da base é o **TerraClimate**, um conjunto de dados climáticos mensais em grade espacial, de cobertura global, amplamente utilizado em pesquisas sobre clima, água, agricultura, vegetação e balanço hídrico.

Referência principal:

- Abatzoglou, J. T., Dobrowski, S. Z., Parks, S. A., e Hegewisch, K. C. *TerraClimate, a high-resolution global dataset of monthly climate and climatic water balance from 1958 onward.*

Links úteis:

- https://www.climatologylab.org/terraclimate.html
- https://climate.northwestknowledge.net/TERRACLIMATE-DATA/

É importante deixar claro que o arquivo publicado aqui **não é uma cópia bruta do TerraClimate**. Trata-se de uma base derivada, processada espacialmente para o território municipal brasileiro.

## Como os dados foram coletados e transformados

O processo começou com o download dos arquivos anuais do TerraClimate em formato NetCDF. Cada arquivo contém 12 camadas mensais de uma variável climática para um determinado ano. O projeto utiliza essas grades como fonte primária e, em seguida, faz a agregação espacial para os municípios do Brasil.

O recorte territorial dos municípios foi obtido a partir da malha municipal em GeoJSON utilizada no projeto. Essa malha foi normalizada em `EPSG:4326`, o mesmo sistema de coordenadas geográficas assumido para o processamento do TerraClimate. Antes do cálculo municipal, os rasters globais foram limitados à área do Brasil, o que reduz custo computacional e evita processamento desnecessário fora da área de interesse.

Para cada variável e para cada ano disponível no período configurado, o processamento percorre os 12 meses do raster mensal. Em seguida, calcula um valor municipal a partir da relação espacial entre os pixels climáticos e o polígono de cada município.

O método principal utilizado foi a **média dos pixels do raster contidos dentro do polígono municipal**. Em outras palavras, quando um município intercepta adequadamente a grade climática, o valor atribuído ao município é a média espacial dos pixels que caem em seu território. Essa é a etapa central que transforma uma base em grade espacial em uma base municipal tabular.

Há, no entanto, municípios em que esse procedimento pode falhar ou se tornar pouco estável, principalmente em casos de geometrias pequenas, estreitas ou com baixa incidência efetiva de pixels na escala da grade. Nessas situações, o script utiliza um **fallback por centroide**. Isso significa que, quando a média espacial por polígono não retorna valor válido, o sistema busca o valor do pixel mais próximo ao centroide do município e o utiliza como substituto. Esse mecanismo evita deixar o município sem informação por uma limitação geométrica do encontro entre grade e polígono.

Depois de obter o valor municipal de cada mês em cada ano, o script não publica diretamente os dados anuais. Em vez disso, ele acumula os valores por mês ao longo do período histórico e calcula uma média climatológica mensal. Assim, o valor final de `ppt_jan`, por exemplo, não é a chuva de um único janeiro, mas a média de todos os janeiros disponíveis no período processado. O mesmo raciocínio vale para temperatura, evapotranspiração e déficit hídrico.

No caso específico da temperatura média, ela não foi lida diretamente de um campo bruto do TerraClimate com esse nome. A variável foi derivada da seguinte forma:

```text
temp_media = (tmin + tmax) / 2
```

Portanto, a temperatura média publicada nesta base representa a média entre a temperatura mínima mensal histórica e a temperatura máxima mensal histórica já agregadas no processo.

## Como a temperatura foi atribuída a cada município

Esse é um ponto central para quem pretende publicar ou reutilizar a base. A temperatura de um município, neste arquivo, **não é a leitura de uma estação meteorológica municipal** e também **não é uma observação pontual feita no centro urbano**. O valor decorre da agregação de uma grade climática espacial.

Na prática, o município recebe um valor climático mensal com base nos pixels do TerraClimate que cobrem o seu território. Se o polígono municipal intercepta pixels suficientes, usa-se a média espacial desses pixels. Se isso não for possível, recorre-se ao valor do pixel mais próximo ao centroide do município. Esse procedimento é metodologicamente defensável para uma base nacional de grande escala, mas precisa ser interpretado corretamente: trata-se de uma **estimativa climática municipal derivada de grade**, e não de uma medição local direta.

Esse cuidado é especialmente importante em municípios muito pequenos, litorâneos, montanhosos ou com forte heterogeneidade interna. Nesses casos, o valor municipal é útil para análise comparativa e estudos territoriais amplos, mas não substitui observações locais detalhadas.

## Importância da base

O principal valor deste arquivo está em tornar operacional uma informação que normalmente está dispersa em formatos difíceis de usar em aplicações municipais. Muitas fontes climáticas robustas existem em formatos raster, NetCDF ou séries por estação. Essas fontes são excelentes para pesquisa e modelagem, mas nem sempre são amigáveis para equipes que precisam trabalhar com recortes administrativos, tabelas, dashboards, APIs e cruzamentos com bases públicas municipais.

Ao transformar a climatologia histórica do TerraClimate em uma base municipal com `codigo_ibge`, o arquivo facilita:

- cruzamento com população, PIB, renda, IDHM e outros indicadores do município;
- análise territorial comparativa em escala nacional;
- elaboração de mapas coropléticos;
- estudos de sazonalidade climática;
- construção de indicadores compostos de vulnerabilidade ou exposição;
- apoio a planejamento público, pesquisa aplicada e visualização de dados.

Esse tipo de estrutura também é útil para produtos digitais. Em vez de carregar rasters pesados ou processar NetCDF em tempo de execução, uma aplicação pode consumir diretamente um CSV municipal leve e responder rapidamente a consultas por município, UF, variável e mês.

## O que o dado pode e o que ele não pode representar

Esta base é adequada para responder perguntas como:

- quais municípios têm janeiro historicamente mais quente?
- como a precipitação média de março varia entre regiões?
- quais áreas combinam altas temperaturas e alto déficit hídrico?
- como a sazonalidade climática municipal se relaciona com agricultura, água ou vulnerabilidade social?

Por outro lado, esta base não deve ser usada como prova de:

- condição meteorológica em tempo real;
- temperatura observada em uma data específica;
- comportamento microclimático de bairro, serra, vale ou área urbana muito localizada;
- substituição de laudos meteorológicos, hidrológicos ou agroclimáticos especializados.

## Ressalvas metodológicas

As principais ressalvas são as seguintes.

Primeiro, o TerraClimate é uma base em grade espacial. Isso significa que o valor municipal publicado aqui depende da resolução do dado original e da forma como essa grade intercepta o polígono do município. Embora seja uma abordagem sólida para análises em larga escala, ela não captura integralmente variações muito finas do relevo, da urbanização ou do uso do solo.

Segundo, a base publicada é uma **climatologia histórica mensal**, e não uma série temporal completa ano a ano. O foco aqui é oferecer um resumo estável do comportamento climático típico dos meses do ano, e não uma reconstrução cronológica detalhada de cada evento climático.

Terceiro, municípios muito pequenos podem depender do fallback por centroide quando a média por polígono não encontra pixels válidos suficientes. Isso não invalida o dado, mas exige cautela adicional na interpretação local.

Quarto, a temperatura média foi derivada de `tmin` e `tmax`, e não necessariamente obtida de um campo bruto com esse nome. O mesmo raciocínio vale para a leitura do arquivo como um todo: ele é um produto derivado, não um dump bruto do fornecedor original.

Quinto, a cobertura histórica efetiva depende da disponibilidade dos arquivos anuais processados. Se algum ano estiver ausente na pasta de entrada, a média mensal será calculada com os anos realmente disponíveis.

## Estrutura do arquivo

Os campos estruturais do CSV são:

- `codigo_ibge`
- `nome_municipio`
- `uf`
- `periodo_inicio`
- `periodo_fim`
- `fonte_clima`
- `metodo_processamento`

Em seguida, o arquivo contém os blocos mensais para:

- `temp_media_*`
- `tmin_*`
- `tmax_*`
- `ppt_*`
- `pet_*`
- `aet_*`
- `def_*`

Essa estrutura foi pensada para ser simples de consumir em Python, R, SQL, JavaScript, BI e ferramentas de visualização.

## Conclusão

O arquivo `clima_historico_municipal.csv` foi criado para tornar mais acessível um tipo de informação que costuma existir em formatos técnicos pouco imediatos para uso municipal. Seu valor está justamente em transformar dados climáticos em grade em uma base municipal interoperável, com metodologia explícita, rastreabilidade da fonte e estrutura adequada para análise territorial em escala nacional.

Ele deve ser entendido como um produto analítico derivado, útil para comparação, integração e visualização, desde que interpretado com as ressalvas apropriadas quanto à resolução espacial, ao uso de médias históricas mensais e ao procedimento de agregação por município.

