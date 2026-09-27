# Documento de Exploração e Ideação: Sistema IoT de Monitoramento Agrícola

## 1. Visão Geral
Este documento detalha a fase inicial de levantamento de requisitos, análise de cenários e ideação para o desenvolvimento de um Sistema IoT de Monitoramento Agrícola focado em eficiência hídrica e automação de irrigação.

## 2. Tree of Thoughts (Árvore de Pensamentos)
A exploração utilizou a metodologia Tree of Thoughts para diversificar os caminhos de solução:
- **Objetivo Principal:** Maximizar a eficiência hídrica e produtividade agrícola através da utilização de sensores de solo.
- **Ramo 1: Previsibilidade (Clima):** Cruzar dados de umidade do solo com a previsão do tempo para bloquear irrigações desnecessárias.
- **Ramo 2: Espacialização (Malha IoT):** Dividir a área rural em zonas de manejo utilizando malhas de sensores sem fio (ex: LoRaWAN) para realizar irrigação setorizada de alta precisão.
- **Ramo 3: Nutrição (Fertirrigação):** Utilização de sensores de Eletrocondutividade (EC) e pH para injetar nutrientes na água de forma autônoma.

## 3. Casos de Uso (UML Textual)
Modelagem do comportamento lógico do sistema e a interação com os atores envolvidos.

**Atores:**
- **Agricultor/Agrônomo:** Ator primário humano.
- **Sistema Atuador:** Controladores de bombas e válvulas de irrigação.
- **Cronômetro/Tempo:** Ator externo autônomo (responsável por triggers baseados em agendamento).
- **API Climática:** Ator externo de fornecimento de dados ambientais.

**Casos de Uso Principais:**
- **UC01: Avaliar Necessidade de Irrigação**
  - **Ator Primário:** Cronômetro/Tempo (Desperta o sistema em intervalos regulares).
  - **Relacionamento:** `<<include>>` **UC02: Ler Sensores de Solo**. Sempre que o cronômetro aciona o sistema para avaliar a irrigação, a leitura física dos parâmetros de umidade e temperatura do solo é executada obrigatoriamente.
- **UC03: Monitorar Clima e Ambiente**
  - **Ator Primário:** Sistema / API Climática
  - **Relacionamento:** `<<extend>>` **UC04: Emitir Alerta Meteorológico**. Caso a API retorne previsões de condições extremas (ex: volume altíssimo de chuva ou risco de geada), o fluxo-base é estendido para emitir um alerta ao Agricultor e aplicar uma trava de segurança ou ação preventiva na bomba.
- **UC05: Executar Ciclo de Irrigação Setorizada**
  - **Ator Primário:** Sistema Atuador. Executa a liberação de água para zonas específicas com base nos dados aprovados pelo UC01 e UC03.

## 4. Triagem, Poda e Solução Definitiva
Durante o processo de ideação, funcionalidades de altíssima complexidade e custo (como monitoramento por Drones/VANTs e injeção iônica de nutrientes em tempo real) foram podadas para concentrar esforços em uma solução robusta, comercializável e escalável.

As soluções mais promissoras (Predição Climática vs. Malha Densa) foram combinadas na arquitetura definitiva:

### Solução Escolhida: A "Malha de Sobrevivência" Macro-Setorizada
- **Conceito (Hardware + Inteligência Preditiva):** A propriedade não é monitorada por uma malha exaustiva de sensores, mas sim dividida em **macrozonas** de acordo com a topografia e o tipo de solo (ex: setor arenoso e setor argiloso). Um sensor-mestre central é instalado em cada macrozona.
- **Mecanismo:** O algoritmo de **Bloqueio Climático (Antecipação de Chuva)** atua de forma independente e customizada para cada zona.
- **Vantagens Competitivas:** 
  1. Reduz drasticamente o custo inicial e os pontos de falha no campo (manutenção facilitada).
  2. Implementação hidráulica viável (poucos setores macro).
  3. Evita desperdícios hídricos com a inteligência climática de software.
  4. Respeita as singularidades do terreno e previne afogamento de raízes em áreas argilosas ou estresse severo em áreas arenosas.
