Você é um especialista em Desenvolvimento e Inovação na área de agronomia.  Preciso que dê ideias sobre um Sistema IoT de Monitoramento agrícola com domínio em sensores de solo e automação de irrigação. Utilize a Tree of Thoughts para requisitos e casos de uso com atores externos (ator tempo/cronometro), <<include>> (ler sensores) e <<extend>> (alerta metereologico). Faça um levantamento de diferentes ideias e sugira-as ranqueando as melhores ideias para que possamos podá-las.

Descarte as ideias 3 e 4, as ideias 1 e 2 são muito boas mas só podemos utilizar apenas uma robusta. Qual as vantagens e desvantagens de cada uma e há mais ideias geradas a partir de ambas?

A derivada B é a melhor opção. Agora que já temos os requisitos, casos de usos e a ideação realizada, documente essa etapa de exploração na pasta docs. Em seguida, faremos a proposta utilizando de parte da SKILL.md como apoio.

Agora será necessário criar 3 diagramas considerando as instruções de SKILL.md e os levantamentos de exploracao-iot-agricola.md. Um diagrama de classes de Composição (Zonairrigacao *-- Valvula), Agregação e multiplicidades. Um DER (Diagrama Entidade Relacionamento) com modelo de dados relacional para leituras agrícolas. Por último, um diagrama de objetos com instância real de uma zona agrícola em irrigação. Essa fase de exploração deve estar dentro de docs em um novo arquivo .md.

Revise o trabalho realizado até o momento considerando os parâmetros iniciais já discutidos. Além disso, verifique a visualização dos diagramas, pois o Antigravity não está renderizando apesar de estarem funcionais em visualizadores externos.

Utilizaremos a técnica de prompt chaining para realizar um modelagem dinâmica de diagramas e tabelas BCE, Sequência, Estados, Atividades e Componentes. Em Suma, o output do primeiro passo (passo A) servirá de input para o próximo passo e assim em diante. Para o primeiro passo (Passo A), faça a tabela BCE e Diagrama de Sequência do acionamento de irrigação. Vamos adicioná-la em um arquivo denominado modelagem-dinamica-iot.md

Para o passo B, utilize essas informações exatas para gerar o Diagrama de Estados do AtuadorValvula (Fechada -> Aberta -> Em Falha) e o Diagrama de Atividades. O diagrama de componentes ficará para o próximo passo (C).

Para finalizar o nosso prompt chaining, considerando as informações exatas do passo anterior, gere o Diagrama de componentes demonstrando a Clean Architecture. Em seguida, planejaremos o Mapeamento Arquitetural e auto-correção.

Antes de prosseguirmos, realize uma checagem das etapas realizadas e o resultado gerado. Em algum momento houve uma fuga do escopo, requisitos, casos de uso ou da proposta do sistema?

Bom sinalizar esse detalhe do estado "Em Falha". Gere um arquivo .md com a justificativa dos planos e pequenos detalhes ou alterações para manter a robustez solicitada.

Agora podemos prosseguir para o Mapeamento Arquitetural (DDD, Clean Architecture e Plano TDD) + Auto-Correção. Primeiro, especifique os Aggregates, Value Objects imutáveis (LeituraUmidade, Coordenada), Repositories e o Domain Service (ValidadorNecessidadeIrrigacao). Após isso, atualize o justificativas-arquiteturais.md.

Você já realizou isso, mas verifique mais uma vez a estrutura de pastas src/ e o Plano de Testes TDD na pasta mapeamento-arquitetural-iot.md. A estrutura de pastas está correta e respeitando a clean architecture? O plano de Testes abrange as ideologias do TDD e está devidamente adequado?

Verifique o arquivo justificativas-arquiteturais.md se está devidamente atualizado. Em seguida, execute o Self-Correction Lopp para validar todas as 15 seções do checklist da skill e das tarefas realizadas (Exploração com zero-shot e alta temperatura com roleplaying e ToT, os diagramas gerados, etc).

Apenas para fins de confirmação e segurança, revise o arquivo documento-requisitos.md se está de acordo com o checklist do arquivo SKILL.md a partir da linha 423.

Mais uma vez, por segurança, vamos revisar os arquivos do projeto, em especial, o documento-requisito.md. O objetivo de elaborar um Software Design Documents (SDD) completo em Markdown com diagramas em Mermaid utilizando LLM através de Engenharia de Prompts (Fase 1 - Explorar e Fase 2 - Propor) foi concluído devidamente? Além disso, a pasta docs/ contém a especificação completa do sistema?

Faça um arquivo de roteiro ou resumo em markdown para servir de guia para explorar os arquivos e compreender a ordem de etapas realizada