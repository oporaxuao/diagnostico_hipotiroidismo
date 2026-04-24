🏥 Diagnóstico de Hipotiroidismo: Quando o Tempo é o Fator Crítico
O hipotiroidismo é uma condição silenciosa e complexa. Quando a glândula tireoide não produz hormônios suficientes, o corpo humano desacelera, resultando em fadiga extrema, depressão e risco aumentado de doenças cardiovasculares.

O diagnóstico preciso e rápido é a fronteira entre uma vida normal e o agravamento severo da saúde de um paciente. Neste projeto, o foco não é apenas prever uma doença, mas reduzir o tempo de espera e evitar complicações através de uma triagem clínica mais inteligente utilizando Machine Learning.

🎯 O Desafio Clínico e a Métrica que Importa
No contexto médico, os erros não têm o mesmo peso. Dizer a uma pessoa saudável que ela precisa de mais exames (Falso Positivo) gera um custo financeiro e um desconforto temporário. No entanto, mandar para casa um paciente doente dizendo que ele está bem (Falso Negativo) pode resultar em complicações graves e irreversíveis a longo prazo.

Por isso, este projeto foi construído com um objetivo inegociável: Minimizar os Falsos Negativos. A estrela guia de toda a modelagem foi a métrica de Recall (Sensibilidade).

🔬 Metodologia e Abordagem
O projeto segue a estrutura CRISP-DM (Cross-Industry Standard Process for Data Mining), garantindo que o entendimento do negócio dite as regras para os dados.

A Base de Dados: Analisamos 3.772 registros de pacientes (Base_M43_Pratique_Hypothyroid.csv), enfrentando desafios comuns do mundo real:

Desbalanceamento Severo: Apenas 7.7% dos casos eram positivos. Modelos comuns ignorariam a doença. Foi necessário ajustar a sensibilidade da modelagem para capturar a classe minoritária.

Dados Ausentes: A conversão cuidadosa de dados faltantes e o tratamento de outliers (como idades registradas acima de 100 anos) garantiram a integridade da análise.

🧠 A Solução e a Explicabilidade (SHAP)
O modelo que apresentou o melhor desempenho para a nossa necessidade crítica (Recall) foi o LightGBM, otimizado através da busca de hiperparâmetros.

Mais do que uma "caixa preta", o modelo precisava falar a língua dos médicos. Utilizando a análise SHAP, confirmamos que o modelo aprendeu a lógica biológica correta:

TSH: Identificado como o sinal clínico mais forte (um TSH elevado é o alarme primário do corpo).

T3 e TT4: Os hormônios diretos apareceram como as variáveis secundárias mais preditivas.

O modelo não apenas prevê; ele corrobora o conhecimento médico estabelecido.

🚀 Impacto e Próximos Passos
Este projeto demonstra como algoritmos avançados podem atuar na linha de frente da saúde, auxiliando médicos na triagem rápida e garantindo que o paciente receba o tratamento correto o mais rápido possível.

Como próximos passos para uma evolução do produto, o modelo deve passar por validação cruzada em um ambiente hospitalar controlado, garantindo sua segurança e eficácia antes de qualquer implementação em produção.
