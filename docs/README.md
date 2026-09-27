# Guia de Leitura: Sistema IoT de Monitoramento Agrícola

Bem-vindo à especificação completa (Software Design Document) do **Sistema IoT de Monitoramento Agrícola**. 

Este projeto foi orquestrado utilizando **Engenharia de Prompts (Prompt Chaining)** em duas grandes macro-fases. Para compreender a evolução arquitetural e garantir que o contexto não se perca, recomendamos fortemente a leitura dos arquivos na seguinte ordem cronológica:

## Fase 1: Exploração e Ideação (Divergência e Convergência)
**1. [Exploração e Ideação](exploracao-iot-agricola.md)**  
*Onde o escopo nasceu.* Utilizamos a técnica *Tree of Thoughts* aliada a um roleplay agronômico para ramificar possibilidades e "podar" ideias irreais para o mercado (como drones e sensores de nutrientes caros). Aqui estabelecemos a arquitetura central de negócio: a **Malha Macro-Setorizada** com inteligência preditiva.

---

## Fase 2: Proposição e Modelagem (A Engenharia de Software)
Com o problema validado, aplicamos o rigor técnico exigido pelas metodologias UML, DDD e Clean Architecture.

**2. [Documento de Requisitos e Rastreabilidade](documento-requisitos.md)**  
*A âncora do projeto.* Define os Requisitos Funcionais (RFs) e Não Funcionais (RNFs). Apresenta o Diagrama de Casos de Uso (mapeando o gatilho autônomo `Cronômetro` e a regra de `<<extend>>` do Alerta Meteorológico). Encerra com a essencial **Matriz de Rastreabilidade** ponta a ponta.

**3. [Diagramas Estruturais](diagramas-estruturais-iot.md)**  
*O esqueleto do sistema.* Contém o Diagrama de Classes (detalhando as relações rígidas de Composição entre Zona e Válvula, e Agregação com Sensores), o DER (para banco de dados relacional) e um snapshot em tempo de execução via Diagrama de Objetos.

**4. [Modelagem Dinâmica](modelagem-dinamica-iot.md)**  
*O comportamento em tempo real.* Construído via *Chaining*, cada diagrama consome os outputs do anterior:
*   Inicia na tabela **BCE (Boundary-Control-Entity)**.
*   Flui para o **Diagrama de Sequência** do acionamento de irrigação.
*   Desdobra o ciclo de vida crítico no **Diagrama de Estados** (Atuador).
*   Descreve a tomada de decisão no **Diagrama de Atividades**.
*   Consolida fisicamente os pacotes no **Diagrama de Componentes**.

**5. [Mapeamento Arquitetural (DDD, Clean Architecture e TDD)](mapeamento-arquitetural-iot.md)**  
*As regras de código-fonte.* Blinda o domínio do sistema através dos padrões táticos do **DDD** (Definição da raiz do Agregado, *Value Objects* imutáveis contra corrupção de dados e *Domain Services* stateless). Detalha as fronteiras inegociáveis da *Clean Architecture* e traça o rigoroso plano metodológico de testes (TDD) para impedir bugs catastróficos.

**6. [Justificativas Arquiteturais (Auto-Correção)](justificativas-arquiteturais.md)**  
*A defesa do Arquiteto.* Registra detalhadamente o *porquê* das decisões cruciais. Justifica a necessidade vital do estado `Em_Falha` da válvula, a adoção do Polling em vez de Push (para preservar a bateria dos sensores no campo) e a utilidade do Bloqueio Climático.

---
💡 **Aviso Técnico:** *Todos os diagramas visuais desta documentação foram desenhados na sintaxe textual **Mermaid**. Eles renderizam nativamente na visualização da sua IDE, no repositório GitHub e na interface de artefatos do Antigravity, eliminando a necessidade de ferramentas externas de UML.*
