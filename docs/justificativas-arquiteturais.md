# Justificativas Arquiteturais e Decisões de Robustez

Este documento registra as razões técnicas por trás das micro-decisões de projeto tomadas durante a modelagem estrutural e dinâmica do Sistema IoT de Monitoramento Agrícola. O objetivo é evidenciar que os detalhes mapeados (como o estado de falha) não são fugas de escopo, mas sim requisitos não-funcionais vitais para a segurança e viabilidade do sistema no mundo real.

## 1. O Estado "Em Falha" (Atuador/Válvula)
- **Detalhe Adicionado:** A válvula não alterna apenas entre `Aberta` e `Fechada`, mas possui o estado de contenção `Em_Falha`.
- **Justificativa de Robustez:** Em sistemas de irrigação hidráulicos, a bomba de água gera altíssima pressão. Se o software manda a válvula abrir, mas ela trava mecanicamente (ou queima um relé), a água não terá por onde sair e a tubulação principal estourará, ou a bomba queimará. O estado "Em Falha" força o software a aguardar um sinal de confirmação (telemetria) da válvula. Caso não receba dentro de um *timeout*, o sistema transita para falhaa, desliga a bomba principal e emite um alerta de resgate.

## 2. A Inversão de Gatilho: Ator "Cronômetro" (Polling vs Push)
- **Detalhe Adicionado:** O fluxo do sistema não é iniciado pelo sensor de solo gritando "estou seco!", mas sim por um Cronômetro central (Scheduler) orquestrando o fluxo.
- **Justificativa de Robustez:** Sensores IoT no campo (LoRaWAN) dependem fortemente de baterias limitadas. Se o sensor enviasse dados ativamente a cada mínima variação (modo *Push*), sua bateria seria drenada em semanas. O ator Cronômetro acorda o sistema em intervalos precisos (ex: a cada 2 horas) e orquestra a coleta de dados (`<<include>> Ler Sensores`), maximizando a vida útil do hardware na lavoura para durar safras inteiras.

## 3. O Bloqueio Climático (`<<extend>>` Alerta Meteorológico)
- **Detalhe Adicionado:** O cruzamento da necessidade física do solo com a previsão do tempo na nuvem.
- **Justificativa de Robustez:** Irrigar um solo pouco antes de uma chuva natural não causa apenas desperdício gigante de água e energia elétrica; agronomicamente, o excesso de água causa a "lixiviação" (lavagem dos nutrientes do solo para os lençóis freáticos, prejudicando a planta). O fluxo de extensão atua como um escudo financeiro e ambiental para o produtor.

## 4. Adoção da Arquitetura Macro-Setorizada (Derivada B)
- **Detalhe Adicionado:** O uso da Composição estrutural (`ZonaIrrigacao *-- Valvula`) com apenas um `SensorMestre` por Zona de Manejo.
- **Justificativa de Robustez:** No campo, equipamentos eletrônicos sofrem com intempéries (sol, chuva, raios) e tráfego intenso de máquinas pesadas (tratores). Uma malha densa (milhares de sensores) exigiria uma equipe dedicada apenas para dar manutenção no hardware. A abordagem Macro-Setorizada reduz o *Hardware Footprint* em 90%, garantindo alta resiliência e foco no algoritmo de inteligência, não na manutenção de cabos rompidos.

## 5. Inversão de Dependências (Clean Architecture e Tabelas BCE)
- **Detalhe Adicionado:** O isolamento das entidades de domínio (`ZonaIrrigacao` e `Valvula`) de qualquer framework de banco de dados ou APIs.
- **Justificativa de Robustez:** Softwares agrícolas duram décadas. É provável que o fornecedor da API de Clima ou o modelo do Banco de Dados mudem ao longo do tempo. O isolamento garantido pelas portas e adaptadores (`ClimaPort`, `RepoPort`) permite que o "coração agronômico" do sistema permaneça intocado caso a tecnologia de infraestrutura seja substituída.

## 6. Auto-Correção e Padrões Táticos DDD
- **Detalhe Adicionado:** Uso estrito de *Value Objects* imutáveis (`LeituraUmidade`, `Coordenada`) e da figura do *Domain Service* (`ValidadorNecessidadeIrrigacao`).
- **Justificativa de Robustez:** 
  - **Imutabilidade (Value Objects):** Protege contra adulteração acidental na memória do sistema. Umidade medida no campo e enviada via IoT pode sofrer latência ou ruído elétrico. Se a leitura de `umidade = -5%` não passar pelas regras nativas do VO (causando exceção no momento da criação), o dado "sujo" sequer passa a existir no sistema.
  - **Isolamento de Complexidade (Domain Service):** A decisão de cruzar se a Zona deve receber irrigação com as métricas do clima externo não pertence somente à Zona. Empilhar *ifs* e condicionais difíceis em classes simples gera código espaguete. Centralizar a inteligência preditiva (Bloqueio Climático) neste Serviço de Domínio agnóstico facilita enormemente o *Plano TDD* garantindo cobertura de 100% de testes unitários sobre a inteligência central do sistema.
