# Previsibilidade_das_Temperaturas_dos_Mancais

#### Aluno: [Gustavo Brito](https://github.com/Gustavo-br-rj)
#### Orientador: [Leonardo Mendonza](https://github.com/leofome8).

---
Trabalho apresentado ao curso [BI MASTER](https://ica.puc-rio.ai/bi-master) como pré-requisito para conclusão de curso e obtenção de crédito na disciplina "Projetos de Sistemas Inteligentes de Apoio à Decisão".

Abaixo seguem os arquivos deste projeto:

- [dataset_C-4451_08001A](dataset_C-4451_08001A.csv) - Dataset com as temperaturas de todos os mancais do equipamento C-4451_08001A.
- [1_AVAL_RN_TEMP_C_4451_08001A](1_AVAL_RN_TEMP_C_4451_08001A.ipynb) - Notebook utilizado na fase de estudos com análise exploratória dos dados, preparação dos dados e testes com a LSTM.
- [2_TESTA_PARAMETROS_RN](2_TESTA_PARAMETROS_RN.ipynb) - Notebook para geração de testes com combinação de parâmetros com o objetivo de identificar a melhor configuração para a LSTM.
- [3_SALVA_MODELOS](3_SALVA_MODELOS.ipynb) - Notebook para geração do modelo de LSTM com base nos melhores parâmetros encontrados para cada Mancal.
- [4_PREVER_TEMP_MANCAL](4_PREVER_TEMP_MANCAL.ipynb) - Notebook para previsão das próximas temperaturas com base no melhor modelo LSTM encontrado para cada mancal.

---

### Resumo

A Transpetro é uma empresa de logística da Petrobras responsável pelo transporte (dutoviário, rodoviário e por navios) e armazenamento de óleo, gás e derivados. A realização do transporte através de dutos requer a manutenção de diversos equipamentos, dentre eles, bombas, geradores, dutos, etc. Para que se mantenha a eficiência e segurança dos serviços, a Transpetro possui uma área de manutenção industrial que é responsável por garantir a saúde desses equipamentos e uma das avaliações executadas é o controle de temperatura dos mancais desses equipamentos.

Este trabalho visa prever a temperatura dos mancais com alguns dias de antecedência de forma a permitir uma análise prévia da saúde dos equipamentos.  

### 1. Introdução

A Transpetro é uma empresa de logística da Petrobras responsável pelo transporte (dutoviário, rodoviário e por navios) e armazenamento de óleo, gás e derivados. A realização do transporte através de dutos requer a manutenção de diversos equipamentos, dentre eles, bombas, geradores, dutos, etc. Para que se mantenha a eficiência e segurança dos serviços, a Transpetro possui uma área de manutenção industrial que é responsável por garantir a saúde desses equipamentos.

Dentro da área de manutenção industrial, existe uma equipe direcionada para o acompanhamento da situação dos equipamentos críticos, que são equipamentos onde a parada pode trazer perdas financeiras relevantes. O acompanhamento da saúde desses equipamentos é feito através da verificação de diversas variáveis coletadas por sistema supervisório aliado a sensores e/ou por meio de atividades de medição realizadas por equipes técnicas em campo.

Dentre os equipamentos acompanhados por esta área, os motores, as bombas e as turbinas possuem sensores para medir a temperatura de seus mancais.

O que é MANCAL?

Os mancais são componentes de máquinas que tem como função, suportar cargas aplicadas em seu próprio eixo, enquanto ele gira. Sendo, portanto, utilizado como parte de sistemas que precisam transferir torque e rotação. Os mancais são compostos por uma estrutura fixa, denominada base, que possui um furo por onde passa um eixo. Desta forma, o elemento proporciona sustentação ao eixo, permitindo que ele gire e mantenha-se sempre em sua posição original.

Atualmente é feito o acompanhamento das temperaturas do dia anterior de funcionamento desses equipamentos. O objetivo é detectar mudanças significativas no padrão comportamental das temperaturas dos mancais que possam indicar a necessidade de intervenção no equipamento para uma manutenção preventiva.

As temperaturas do dia anterior são coletadas à 00:30 através de uma API REST disponibilizada pelo fornecedor do sistema supervisório (General Electric). Após a coleta, são expurgadas todas as medições de temperatura do período em que o equipamento se encontrava desligado, ou seja, apenas são consideradas as medições de temperatura de equipamentos ligados.

Com base nas medições de temperatura do dia anterior (temperatura a cada 5 minutos), é realizada a média do dia para cada mancal e em seguida armazenada em uma base de dados. Após o registro dessas médias os seguintes alertas são enviados por email:

   * Desvio de 40% na temperatura do mancal em relação a média de todos mancais do equipamento no mesmo dia.<br/>
   * Desvio de 20% na temperatura do mancal em relação a sua média semestral.

#### Objetivo

Como foi apresentado acima, a média das temperaturas dos mancais é obtida apenas um dia após as medições realizadas pelos sensores. O objetivo deste trabalho é prever a temperatura desses mancais alguns dias a frente, assim os avisos poderiam ser disparados com antecedência, permitindo acionar com brevidade as equipes de manutenção que irão realizar a verificação em campo desses equipamentos.

Além de contribuir com o processo de manutenção industrial da Transpetro, pretendo com este projeto revisitar algumas das matérias ministradas no curso BI Master, são elas:

  * LUI  - Localização do Uso da Informação<br/>
  * CONF - Confiabilidade<br/>
  * MEAD - Métodos Estatísticos<br/>
  * DM   - Data Mining<br/>
  * RN   - Redes Neurais

### 2. Modelagem

#### 2.1. Extração dos dados do sistema supervisório

  Para extração dos dados no sistema supervisário, utilizamos a interface de serviço REST disponibilizada pelo sistema. As extrações são realizadas diariamente em busca dos valores medidos no dia antetior. Para este projeto os valores a serem utilizados são:

   * o **estado do mancal**, que indica se o mancal está ativo ou inativo e;
   * a **temperatura do mancal**.

A API REST utilizada para obteção dos valores acima é:
  
  https://<historianservername>:8443/historian-rest-api/v1/datapoints/interpolated?tagNames=[tagsName]&amp;start=[ontem-yyyy-MM-dd]T00:00:00.111Z&amp;end=[ontem-yyyy-MM-dd]T23:59:59.111Z&amp;count=0&amp;intervalMs=300000&amp;quality=3
  
onde,
  
  **[tagsName]** é o conjunto de tags no sistema supervisório, separdos por ";", que identificam os sensores a serem observados. Para este projeto utilizaremos os sensores de temperatura dos mancais dos equipamentos.
  
  **[ontem-yyyy-MM-dd]** é a data do dia anterior a execução do extrator de dados.
  
**Observação:** o parâmetro **intervalMs=300000** indica que o serviço REST trará o valor medido para cada intervalo de 5 seg. 
  
#### 2.2. Cálculo da média do dia

  Com base nos resultados da extração para cada mancal, é verificado se em cada tempo de leitura (a cada 5 seg.) o estado do mancal é inativo, caso esteja inativo, aquela leitura de temperatura do mesmo instante é desconsiderada.
  
  Ao final do processo de expurgo das temperaturas nos instantes em que um mancal se encontra inativo, é feita a média de temperatura do dia do mancal. Por exemplo:
  
  DATA/HORA           | Equipamento    | Mancal (Tag)          | Estado  | Temperatura
  --------------------|----------------|-----------------------|---------|-------------
  2021-06-12T00:00:05 | GE-4150.01001B | ECE1.G2-TI-MANC3.F_CV | ATIVO   | 50,00
  2021-06-12T00:00:10 | GE-4150.01001B | ECE1.G2-TI-MANC3.F_CV | ATIVO   | 53,20
  2021-06-12T00:00:15 | GE-4150.01001B | ECE1.G2-TI-MANC3.F_CV | ATIVO   | 52,20
  2021-06-12T00:00:20 | GE-4150.01001B | ECE1.G2-TI-MANC3.F_CV | ATIVO   | 51,60
  2021-06-12T00:00:25 | GE-4150.01001B | ECE1.G2-TI-MANC3.F_CV | ATIVO   | 55,20
  **2021-06-12T00:00:30** | **GE-4150.01001B** | **ECE1.G2-TI-MANC3.F_CV** | **INATIVO** | **30,00**
  **2021-06-12T00:00:35** | **GE-4150.01001B** | **ECE1.G2-TI-MANC3.F_CV** | **INATIVO** | **34,00**
  2021-06-12T00:00:40 | GE-4150.01001B | ECE1.G2-TI-MANC3.F_CV | ATIVO   | 54,00
  
  Neste caso, a média da temperatura do dia **12/06/2021** para o mancal **ECE1.G2-TI-MANC3.F_CV** do equipamento **GE-4150.01001B** não consideraria as entradas dos tempos 00:00:30 e 00:00:35.
  
  Após a média ser gerada, o resultado é gravado na tabela **SCDA_PM_AVGVAL_TAG** de um banco de dados Oracle para que em seguida seja verificado através de script SQL se para algum mancal as condições abaixo se apresentam: 

  * Desvio de 40% na temperatura do mancal em relação a média de todos mancais do equipamento no mesmo dia.<br/>  
  * Desvio de 20% na temperatura do mancal em relação a sua média semestral.

  Caso alguma das condições aconteça, um email é enviado para os responsáveis pelo acompanhamento do equipamento.
  
#### 2.3. Consulta à base de dados com as médias do dia de cada mancal

  Neste ponto será feita a leitura da base de dados para que possamos aplicar uma inteligência artificial que nos permita prever a temperatura dos mancais alguns dias a frente.

  A leitura a base de dados é feita da seguinte forma:
  
**SELECT** A.SCDA_CD_EQUIP AS EQUIPAMENTO,</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A.SCDA_CD_TAG AS MANCAL,</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TO_CHAR(A.SCDA_DT_VALOR, 'dd/mm/yyyy') AS DATA,</br> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;REPLACE( TO_CHAR( A.SCDA_NR_AVG_VAL ), ',', '.') AS TEMPERATURA</br>
**FROM** SCDA_PM_AVGVAL_TAG A **INNER JOIN** SCDA_PM_TAG_EQUIP B</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**ON** A.SCDA_CD_EQUIP = B.SCDA_CD_EQUIP</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**AND** A.SCDA_CD_TAG   = B.SCDA_CD_TAG</br>
**WHERE** B.SCDA_CD_TPTAG **LIKE** 'TMP_MANC_%'</br> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**AND** B.SCDA_CD_GRTAG = 'CTMP'</br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**AND** A.SCDA_CD_EQUIP **IN**  ( 'C-4451.08001A' )</br>
**ORDER BY** A.SCDA_CD_EQUIP, A.SCDA_CD_TAG, A.SCDA_DT_VALOR
  
**Observação:** para este estudo, utilizaremos o equipamento **C-4451.08001A**.

O conjunto de dados que iremos trabalhar neste projeto terá o seguinte formato:

EQUIPAMENTO   | MANCAL           | DATA       | TEMPERATURA
--------------|------------------|------------|--------------
C-4451.08001A | EJA1.A-TC21.F_CV | 04/01/2021 |	659.03825
C-4451.08001A | EJA1.A-TC21.F_CV | 05/01/2021	| 659.76301
C-4451.08001A | EJA1.A-TC21.F_CV | 06/01/2021	| 649.72128
C-4451.08001A | EJA1.A-TC21.F_CV | 07/01/2021	| 657.68837
C-4451.08001A | EJA1.A-TC21.F_CV | 08/01/2021	| 660.11167
C-4451.08001A | EJA1.A-TC21.F_CV | 09/01/2021	| 660.8411
C-4451.08001A | EJA1.A-TC21.F_CV | 10/01/2021	| 663.60157


#### 2.4. Análise exploratória e preparação dos dados

A análise exploratória dos dados e a preparação podem ser encontradas na parte 1 e 2 do arquivo notebook [1_AVAL_RN_TEMP_C_4451_08001A](1_AVAL_RN_TEMP_C_4451_08001A.ipynb).

Durante a análise exploratória obtivemos informações estatísticas sobre a base de dados trabalhada, como por exemplo, temos abaixo algumas estatísticas sobre as temperaturas dos mancais do equipamento C-4451.08001A:

MEDIDA|TEMPERATURA
------|------------
count	|2245.000000
mean	|64.265038
std 	|16.315631
min	  |18.718320
25%	  |54.898560
50%	  |59.504080
75%	  |71.582280
max	  |390.441750

O problema a ser solucionado trata-se de uma previsão de série temporal, onde precisaremos identificar quais as temperaturas dos mancais nos próximos x dias. Para esse problema utilizaremos a arquitetura de redes neurais LSTM (Long Short Term Memory). 

**Obs.:** para utilização dos arquivos notebook será necessário criar a seguinte estrutura de pasta no Google Drive:

- **Meu Drive/BI_MASTER/PROJETO_FINAL_BIMASTER/DATASETS** - Diretório onde se encontra o arquivo com os dados de temperatura dos mancais.
- **Meu Drive/BI_MASTER/PROJETO_FINAL_BIMASTER/AVALIACAO_REDES** - Diretório com o resultado das avaliações das RNs geradas com os diversos parâmetros através do notebook [2_TESTA_PARAMETROS_RN](2_TESTA_PARAMETROS_RN.ipynb).  
- **Meu Drive/BI_MASTER/PROJETO_FINAL_BIMASTER/MODELOS** - Diretório onde ficarão os modelos gerados com base nos parâmetros que tiveram as melhores avaliações. Este modelos são gerados através do notebook [3_SALVA_MODELOS](3_SALVA_MODELOS.ipynb).

Após a etapa de análise exploratória e preparação dos dados, foram aplicados alguns parâmetros em uma rede neural do tipo LSTM para testes sobre o conjunto de dados referente às temperaturas dos mancais do equipamento **C-4451.08001A**. Dentre os parâmetros testados, ficou bastante evidente que os optmizers **Adam e Adadelta** forneciam uma previsão mais próxima dos valores reais.

### 3. Resultados

#### 3.5. Avaliação dos parâmetros da rede LSTM

Dado que o processo de teste individual dos diversos parâmetros da rede se demonstrou muito custoso, foi criado o notebook [2_TESTA_PARAMETROS_RN](2_TESTA_PARAMETROS_RN.ipynb) para geração dos testes baseados na combinação dos seguintes parâmetros adotados:

* windowSet   = [3,4,5,6]
* outputSet   = [2,3]
* layerSet    = [1,2,3]
* unitSet     = [[150,80,80],[120,60,40],[80,80,60]]
* dropoutSet  = [0.2, 0.25, 0.3, 0.35] 
* optmizerSet = ['Adam', 'Adadelta']
* epochs      = 800
* batch_size  = 32

A combinação desses parâmetros gerou **576 configurações** que foram aplicados à rede neural LSTM. Estes testes foram realizados para cada um dos mancais.

Ao final da execução das 576 configurações para cada umm dos mancais, um arquivo do tipo CSV é gerado para cada mancal com o resultado de cada configuração, apresentado as métricas de avaliação **MSE, RMSE e MAPE**. O nome do arquivo segue o padrão **[NOME_EQUIPAMENTO]__[NOME_MANCAL]_results.csv**, por exemplo, C-4451.08001A__EJA1.A-TE1209B.F_CV_results.csv.

Métricas de avaliação utilizadas:
- **MSE** - Mean Squared Error
- **RMSE** - Root Mean Squared Error
- **MAPE** - Mean Absolute Percentage Error

**Obs.:** por conta do tempo de processamento necessário para realizar os testes de cada combinação de parâmetros, foi necessário adquirir o plano básico do Google Colab para que a aplicação pudesse rodar durante a madrugada e fazendo uso de GPU. A versão gratuita do Google Colab reiniciava o serviço sempre às 0:00, impedindo a conclusão dos testes.

#### 3.6. Aplicação dos parâmetros selecionados para a rede LSTM

Com base nas métricas de avaliação das RNs geradas para cada mancal, o notebook [3_SALVA_MODELOS](3_SALVA_MODELOS.ipynb) verifica qual foi a melhor configuração para o mancal avaliado e então cria a RN com esta configuração para em seguida salvar o modelo produtivo no diretório **Meu Drive/BI_MASTER/PROJETO_FINAL_BIMASTER/MODELOS**.

O modelo é criado em uma pasta com o padrão de nomenclatura **[NOME_EQUIPAMENTO]__[NOME_MANCAL]_model** que fica disponível para utilização pelo notebook [4_PREVER_TEMP_MANCAL](4_PREVER_TEMP_MANCAL.ipynb)

#### 3.7. Verificação de desvios de temperatura

Após cada mancal ter seu modelo produtivo da LSTM gerado, o notebook [4_PREVER_TEMP_MANCAL](4_PREVER_TEMP_MANCAL.ipynb) faz o carregamento desses modelos e aplica a previsão das temperaturas dos próximos 3 dias para cada mancal e faz a verificação dos testes abaixo:

  * Desvio de 40% na temperatura do mancal em relação a média de todos mancais do equipamento no mesmo dia.
  * Desvio de 20% na temperatura do mancal em relação a sua média semestral.

### 4. Conclusões

Neste trabalho objetivou-se pela apresentação de uma inteligência artificial capaz de oferecer previsibilidade sobre a temperatura dos mancais, pretendendo-se com isso a identificação de possíveis problemas nos equipamentos com maior antecedência.

Na primeira parte do trabalho contextualizamos o problema e em seguida abordamos a fonte de dados necessária para ser utilizada em uma solução de IA.

Realizamos em seguida uma análise exploratória dos dados visando conhecimento de algumas características estatísticas da base e também de volumetria e qualidade do dado. 

Após a análise dos dados, iniciou-se o processo de busca por melhores parâmetros da RN LSTM que conseguissem representar melhor a variação dos dados reais. Neste ponto o processo de verificação dos parâmetros tornou-se muito custoso e acabou sendo necessário criar um programa para realização da testagem dos diversos parâmetros.

Com base nos melhores parâmetros identificados, os modelos produtivos da RN de cada mancal foram armazenados para utilização pelo programa responsável pela verificação dos desvios de temperatura.

Ao longo do trabalho pudemos observar o comportamento da temperatura dos mancais do equipamento **C-4451.08001A**, e ao aplicar os modelos produtivos das RNs para estes mancais, observamos desvios em 4 deles. 

Será necessário manter um período de observação dessas redes neurais para avaliação de sua performance frente aos dados reais. Após validada esta prova de conceito, deverá ser preparada uma estrutura produtiva que contemple desde a leitura dos dados até a entrega visual dos resultados, e a aplicação do rollout para os demais equipamentos.  

---

Matrícula: 192.110.142

Pontifícia Universidade Católica do Rio de Janeiro

Curso de Pós Graduação *Business Intelligence Master*
