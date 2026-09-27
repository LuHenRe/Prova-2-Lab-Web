# Modelagem Dinâmica: Sistema IoT de Monitoramento Agrícola

Este documento acompanha a construção evolutiva da arquitetura do sistema utilizando a técnica de prompt chaining. A saída de cada etapa servirá de base estrutural para a modelagem da etapa seguinte.

## Passo A: Boundary-Control-Entity (BCE) e Diagrama de Sequência
O foco desta primeira etapa é mapear o fluxo do caso de uso mais crítico do sistema: **Avaliar Necessidade e Acionar Irrigação**.

### 1. Tabela BCE (Fronteira, Controle, Entidade)
A tabela abaixo reclassifica as classes geradas na etapa de exploração para preparar a adoção da Clean Architecture.

| Caso de Uso | Boundary (Fronteira) | Control (Casos de Uso/Aplicação) | Entities (Domínio/Infraestrutura) |
|-------------|----------------------|----------------------------------|-----------------------------------|
| Avaliar Necessidade e Acionar Irrigação | `CronometroBoundary` (Gatilho In), `ClimaAdapter` (Saída API) | `AvaliarIrrigacaoUseCase` | `ZonaIrrigacao`, `SensorMestre`, `LeituraSolo`, `Valvula` |

### 2. Diagrama de Sequência: Acionamento de Irrigação
O diagrama abaixo utiliza o output da tabela BCE para demonstrar a ordem cronológica e síncrona das mensagens na execução do fluxo autônomo. O gatilho temporal orquestra a leitura, valida o clima, aciona a entidade de domínio e salva o log de persistência.

```mermaid
sequenceDiagram
    actor Cronometro
    participant B as CronometroBoundary «boundary»
    participant C as AvaliarIrrigacaoUseCase «control»
    participant S as SensorMestre «entity»
    participant GW as ClimaAdapter «boundary_out»
    participant Z as ZonaIrrigacao «entity»
    participant V as Valvula «entity»
    participant R as IrrigacaoRepository «repository»

    Cronometro ->> B: tick(zonaId)
    B ->> C: executarAvaliacao(zonaId)
    
    %% <<include>> Ler Sensores
    C ->> S: capturarLeitura()
    S -->> C: LeituraSolo (ex: umidade=11%)
    
    %% Avaliar Clima Externo
    C ->> GW: buscarPrevisao(coordenadas)
    GW -->> C: PrevisaoClima (chuva=0mm)
    
    %% Delegação para a Entidade de Domínio (DDD)
    C ->> Z: avaliarNecessidade(LeituraSolo, PrevisaoClima)
    Z -->> C: necessita = true
    
    alt necessita == true
        C ->> Z: iniciarCiclo()
        Z ->> V: abrir()
        V -->> Z: status=aberta
        Z -->> C: cicloIniciado
        
        C ->> R: salvarTransacao(LeituraSolo, ValvulaStatus)
        R -->> C: persistido
    else necessita == false
        C ->> R: salvarLeituraSemAcionamento(LeituraSolo)
        R -->> C: persistido
    end
    
    C -->> B: ok
```

## Passo B: Diagrama de Estados e Diagrama de Atividades
Com base nas decisões orquestradas pelo `AvaliarIrrigacaoUseCase` no Passo A, modelamos o ciclo de vida do nosso componente atuador e o fluxo algorítmico do sistema.

### 1. Diagrama de Estados: AtuadorValvula
A válvula possui um ciclo de vida finito, transitando entre três estados. O estado "Em Falha" é crucial para acionar alertas de hardware e evitar que a bomba hidráulica queime ou que a plantação sofra danos.

```mermaid
stateDiagram-v2
    [*] --> Fechada
    
    Fechada --> Aberta : acionar_abertura() [sem bloqueio climático]
    Aberta --> Fechada : acionar_fechamento() / timeout_ciclo_atingido
    
    Fechada --> Em_Falha : timeout_resposta / sem_comunicacao
    Aberta --> Em_Falha : erro_hidraulico / pane_eletrica
    
    Em_Falha --> Fechada : reset_manual_operador()
```

### 2. Diagrama de Atividades: Orquestração do Acionamento
Este fluxo detalha a inteligência de controle mapeada no Diagrama de Sequência (condicionais if/else "alt"). Ele demonstra os "branches" de decisão baseados na predição climática e na leitura física.

```mermaid
flowchart TD
    Start((Início)) --> A[Gatilho Temporal Disparado]
    A --> B[Ler Umidade do Sensor de Solo]
    
    B --> C{Umidade < Ponto de Murcha?}
    C -- Não --> D[Registrar Leitura Histórica]
    D --> End((Fim))
    
    C -- Sim --> E[Consultar Previsão Climática]
    E --> F{Previsão de Chuva Elevada?}
    
    F -- Sim --> G[Ativar Bloqueio Climático]
    G --> D
    
    F -- Não --> H[Comandar Abertura da Válvula]
    H --> I{Válvula Respondeu Aberta?}
    
    I -- Sim --> J[Registrar Transação de Irrigação]
    J --> End
    
    I -- Não --> K[Transitar Válvula para 'Em Falha']
    K --> L[Emitir Alerta ao Agricultor]
    L --> End
```

## Passo C: Diagrama de Componentes (Clean Architecture)
Finalizando a cadeia estrutural, utilizamos o mapa de atores do Passo A e as camadas lógicas (Fronteira, Controle, Entidade) para consolidar fisicamente a **Clean Architecture**. O diagrama evidencia a Regra de Dependência (setas apontando sempre para o núcleo de Domínio) e a Inversão de Dependência via *Ports & Adapters*.

```mermaid
flowchart TB
    subgraph Infra["Frameworks, Hardware & Drivers"]
        Cron[Scheduler/Timer OS]
        API[API de Clima Externa]
        DB[(PostgreSQL)]
    end
    
    subgraph Adapters["Interface Adapters"]
        Trigger[CronometroBoundary]
        ClimaImpl[ClimaAdapterImpl]
        RepoImpl[IrrigacaoRepositoryPostgres]
    end
    
    subgraph Application["Application (Use Cases)"]
        UseCase[AvaliarIrrigacaoUseCase]
    end
    
    subgraph Domain["Domain (Entities & Ports)"]
        Zona[ZonaIrrigacao «entity»]
        Valvula[Valvula «entity»]
        Sensor[SensorMestre «entity»]
        ClimaPort[[ClimaAdapter «interface»]]
        RepoPort[[IrrigacaoRepository «interface»]]
    end

    %% Interação Infra <> Adapters
    Cron --> Trigger
    ClimaImpl -.consumes.-> API
    RepoImpl -.persists.-> DB

    %% Regra de Dependência (De Fora para Dentro)
    Trigger --> UseCase
    UseCase --> Zona
    UseCase --> ClimaPort
    UseCase --> RepoPort
    
    %% Inversão de Dependência (Interfaces implementadas por fora)
    ClimaImpl -.implementa.-> ClimaPort
    RepoImpl -.implementa.-> RepoPort
    
    %% Relações Internas do Domínio (Conforme Passo A)
    Zona --> Valvula
    Zona --> Sensor
```
