# Diagramas Estruturais: Sistema IoT de Monitoramento Agrícola

Este documento materializa a modelagem de dados e objetos da solução definida (Malha Macro-Setorizada com Inteligência Preditiva), seguindo as diretrizes estruturais de projeto (DDD e Modelagem Relacional).

## 1. Diagrama de Classes
Este diagrama ilustra a estrutura do domínio, evidenciando a **Composição** (A válvula é parte intrínseca da zona de irrigação e controlada por ela) e a **Agregação** (O sensor mestre está alocado em uma zona, mas tem um ciclo de vida independente, podendo ser trocado de lugar ou substituído).

```mermaid
classDiagram
    class ZonaIrrigacao {
        -id: Long
        -nomeSetor: String
        -tipoSolo: String
        -limiteUmidadePMP: Decimal
        +avaliarNecessidadeIrrigacao() bool
        +iniciarCiclo() void
        +pararCiclo() void
    }
    
    class Valvula {
        -id: Long
        -vazaoLitrosHora: Decimal
        -isAberta: boolean
        +abrir() void
        +fechar() void
    }
    
    class SensorMestre {
        -id: Long
        -macAddress: String
        -bateriaPercent: int
        +capturarLeitura() LeituraSolo
    }
    
    class LeituraSolo {
        -id: Long
        -dataHora: Date
        -umidadePercent: Decimal
        -temperaturaCelsius: Decimal
    }
    
    class PrevisaoClima {
        -dataReferencia: Date
        -volumeChuvaMm: Decimal
        -riscoGeada: boolean
    }

    %% Relações e Multiplicidades
    ZonaIrrigacao "1" *-- "1..*" Valvula : compõe
    ZonaIrrigacao "1" o-- "1" SensorMestre : agrega
    SensorMestre "1" -- "0..*" LeituraSolo : gera
    ZonaIrrigacao "1" ..> "1" PrevisaoClima : consome previsão
```

## 2. Diagrama Entidade-Relacionamento (DER)
O DER foca na camada de persistência. Observe como as classes do diagrama acima foram mapeadas para tabelas relacionais, estabelecendo Chaves Primárias (PK) e Estrangeiras (FK). As instâncias temporais (Previsão do Clima) muitas vezes não são persistidas permanentemente como entidade, servindo apenas para tomada de decisão em tempo real.

```mermaid
erDiagram
    ZONA_IRRIGACAO ||--|{ VALVULA : compoe
    ZONA_IRRIGACAO ||--o| SENSOR_MESTRE : agrega
    SENSOR_MESTRE ||--o{ LEITURA_SOLO : registra

    ZONA_IRRIGACAO {
        bigint id PK
        string nome_setor
        string tipo_solo
        decimal limite_umidade_pmp "Ponto de Murcha Permanente"
    }
    
    VALVULA {
        bigint id PK
        bigint zona_id FK
        decimal vazao_litros_hora
        boolean is_aberta
    }
    
    SENSOR_MESTRE {
        bigint id PK
        bigint zona_id FK
        string mac_address
        int bateria_percent
    }
    
    LEITURA_SOLO {
        bigint id PK
        bigint sensor_id FK
        datetime data_hora
        decimal umidade_percent
        decimal temperatura_celsius
    }
```

## 3. Diagrama de Objetos (Snapshot em Tempo de Execução)
Aqui temos uma instância real ("fotografia") do sistema em operação. O talhão arenoso atingiu uma umidade crítica (abaixo do seu limite) e a válvula correspondente está ativada.

```mermaid
classDiagram
    class setorNorteArenoso {
        <<instance>>
        id = 1
        nomeSetor = "Talhão Norte 01"
        tipoSolo = "Arenoso"
        limiteUmidadePMP = 12.0
    }
    
    class valvulaPrincipalNorte {
        <<instance>>
        id = 105
        isAberta = true
        vazaoLitrosHora = 5000.0
    }
    
    class sensorLoraNorte {
        <<instance>>
        id = 42
        macAddress = "00:1B:44:11:3A:B7"
        bateriaPercent = 88
    }
    
    class leitura13h {
        <<instance>>
        id = 99283
        dataHora = "2026-09-25 13:00"
        umidadePercent = 10.5
        temperaturaCelsius = 29.8
    }

    %% Conexões das instâncias (O objeto valvulaPrincipalNorte SÓ existe contido no setorNorteArenoso)
    setorNorteArenoso *-- valvulaPrincipalNorte
    setorNorteArenoso o-- sensorLoraNorte
    sensorLoraNorte --> leitura13h
```
