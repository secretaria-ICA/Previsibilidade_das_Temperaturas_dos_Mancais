# PROJ-BI_MASTER
Projeto Final do Curso CCE PUC-Rio - BI MASTER

### CENÁRIO

A Transpetro é uma empresa de logística da Petrobras e é responsável pelo transporte (dutoviário, rodoviário e por navios) e armazenamento de
óleo, gás e derivados. A realização do transporte através de dutos requer a manutenção de diversos equipamentos, dentre eles, bombas, geradores,
dutos, etc. Para que se mantenha a eficiência e segurança dos serviços, a Transpetro possui uma área de manutenção industrial que é responsável
por garantir a saúde desses equipamentos.

Dentro da área de manutenção industrial, existe uma equipe direcionada para o acompanhamento da situação dos equipamentos críticos, que são equipamentos
onde a parada pode trazer perdas financeiras relevantes. O acompanhamento da saúde desses equipamentos é feito através da verificação de diversas variáveis
coletadas por sistema supervisório aliado a sensores e/ou por meio de atividades de medição realizadas por equipes técnicas em campo.

Dentre os equipamentos acompanhados por esta área, os motores, as bombas e as turbinas possuem sensores para medir a temperatura de seus mancais.

O que é MANCAL?

Os mancais são componentes de máquinas que tem como função, suportar cargas aplicadas em seu próprio eixo, enquanto ele gira. Sendo, portanto, utilizado como parte de sistemas que precisam transferir torque e rotação. Os mancais são compostos por uma estrutura fixa, denominada base, que possui um furo por onde passa um eixo. Desta forma, o elemento proporciona sustentação ao eixo, permitindo que ele gire e mantenha-se sempre em sua posição original.

Atualmente é feito o acompanhamento das temperaturas do dia anterior de funcionamento desses equipamentos. O objetivo é detectar mudanças significativas no padrão comportamental das temperaturas dos mancais que possam indicar a necessidade de intervenção no equipamento para uma manutenção preventiva.

As temperaturas do dia anterior são coletadas à 00:30 através de uma API REST disponibilizada pelo fornecedor do sistema supervisório (General Electric).
Após a coleta, são expurgadas todas as medições de temperatura do período em que o equipamento se encontrava desligado, ou seja, apenas são consideradas as medições de temperatura de equipamentos ligados.

Com base nas medições de temperatura do dia anterior (temperatura a cada 5 minutos), é realizada a média do dia para cada mancal e em seguida armazenada em uma base de dados. Após o registro dessas médias os seguintes alertas são enviados por email:

   1 - Desvio de 40% na temperatura do mancal em relação a média dos demais mancais do equipamento.<br/>
   2 - Desvio de 20% na temperatura do mancal em relação a sua média semestral

### OBJETIVO

Como foi apresentado acima, a média das temperaturas dos mancais é obtida apenas um dia após as medições realizadas pelos sensores. O objetivo deste trabalho é prever a temperatura desses mancais alguns dias a frente, assim os avisos poderiam ser disparados com antecedência, permitindo acionar com brevidade as equipes de manutenção que irão realizar a verificação em campo desses equipamentos.

Além de contribuir com o processo de manutenção industrial da Transpetro, pretendo com este projeto revisitar algumas das matérias ministradas no curso BI Master, são elas:

LUI  - Localização do Uso da Informação<br/>
CONF - Confiabilidade<br/>
MEAD - Métodos Estatísticos<br/>
DM   - Data Mining<br/>
RN   - Redes Neurais
