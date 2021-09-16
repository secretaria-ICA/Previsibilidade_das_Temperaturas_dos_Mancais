# Previsibilidade das Temperaturas dos Mancais

#### Aluno: [Gustavo Brito](https://github.com/Gustavo-br-rj)
#### Orientador: [Leonardo Mendonza](https://github.com/link_do_github).

---
Trabalho apresentado ao curso [BI MASTER](https://ica.puc-rio.ai/bi-master) como pré-requisito para conclusão de curso e obtenção de crédito na disciplina "Projetos de Sistemas Inteligentes de Apoio à Decisão".

<!-- para os links a seguir, caso os arquivos estejam no mesmo repositório que este README, não há necessidade de incluir o link completo: basta incluir o nome do arquivo, com extensão, que o GitHub completa o link corretamente -->
- [Link para o código](https://github.com/link_do_repositorio). <!-- caso não aplicável, remover esta linha -->

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

### 2. Modelagem (EM CONSTRUÇÃO)

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


#### 4. Análise exploratória e preparação dos dados

### 3. Resultados

#### 3.5. Avaliação dos parâmetros da rede LSTM

#### 3.6. Aplicação dos parâmetros selecionados para a rede LSTM

#### 3.7. Verificação de desvios de temperatura

### 4. Conclusões

---

Matrícula: 192.110.142

Pontifícia Universidade Católica do Rio de Janeiro

Curso de Pós Graduação *Business Intelligence Master*
