# Mapeamento Arquitetural (DDD, Clean Architecture e Plano TDD)

## 1. Mapeamento Tático (Domain-Driven Design)
Com o núcleo das operações validado, segregamos a complexidade utilizando os padrões táticos do DDD. Esta blindagem é fundamental para garantir a estabilidade do sistema IoT.

### Tabela de Agregados e Componentes de Domínio

| Aggregate Root | Entidades Internas | Value Objects (Imutáveis) | Repository (Porta de Saída) |
|----------------|--------------------|---------------------------|-----------------------------|
| `ZonaIrrigacao` | `Valvula`, `SensorMestre` | `LeituraUmidade`, `Coordenada` | `IrrigacaoRepository` (Interface) |

### Detalhamento dos Componentes (Auto-correção Estrutural):
- **Aggregate Root (`ZonaIrrigacao`)**: Representa a nossa Fronteira de Consistência. A `Valvula` e o `SensorMestre` não são expostos publicamente. Para comandar a válvula, o caso de uso chama obrigatoriamente a raiz (ex: `zona.iniciarCiclo()`), protegendo a regra de integridade de hardware.
- **Value Objects Imutáveis**:
  - `LeituraUmidade`: Encapsula o valor numérico percentual da leitura de hardware. Como é imutável, garante a não adulteração dos dados na passagem entre camadas. Validações primitivas ocorrem aqui (ex: lançamento de erro para umidades > 100% ou < 0%).
  - `Coordenada`: Encapsula Latitude e Longitude (GPS) para enviar à API de clima. Não possui ID de persistência.
- **Domain Service (`ValidadorNecessidadeIrrigacao`)**: O Bloqueio Climático é encapsulado neste Serviço de Domínio sem estado (stateless), evitando acoplamento incorreto na Entidade ou na API.
- **Repository (Port)**: Conforme o princípio de Inversão de Dependência, o Domínio possui apenas a *Interface* do repositório. A implementação concreta ficará nas camadas externas.

## 2. Estrutura de Camadas (Clean Architecture)
Após auto-correção e revisão profunda, a estrutura de diretórios foi detalhada para evidenciar perfeitamente as fronteiras da Clean Architecture, em especial o conceito de *Ports & Adapters*. A regra inegociável é: dependências apontam sempre de fora para dentro.

```text
src/
 ├── domain/            <- [Camada Interna - Core] Não importa pacotes externos
 │    ├── entities/     <- ZonaIrrigacao, Valvula
 │    ├── value_objects/<- LeituraUmidade, Coordenada
 │    ├── services/     <- ValidadorNecessidadeIrrigacao
 │    └── ports/        <- IrrigacaoRepository (Interface), ClimaAdapter (Interface)
 │
 ├── application/       <- [Camada de Casos de Uso]
 │    ├── usecases/     <- AvaliarIrrigacaoUseCase
 │    └── dtos/         <- Input e Output Data para cruzar fronteiras
 │
 ├── adapters/          <- [Camada de Interface Adapters]
 │    ├── controllers/  <- CronometroTrigger (Gatilho de entrada via OS)
 │    ├── gateways/     <- ClimaAdapterImpl (Implementa ClimaAdapter port)
 │    └── repositories/ <- IrrigacaoRepositoryPostgres (Implementa IrrigacaoRepository port)
 │
 └── infra/             <- [Camada Externa - Frameworks & Drivers]
      ├── database/     <- Scripts SQL de migração (DDL), ORM Configs
      └── config/       <- Setup de Injeção de Dependência (DI) e Cloud
```

## 3. Plano de Testes (Metodologia TDD)
Para garantir zero falhas logísticas (evitando perdas reais na lavoura), a construção do código seguirá rigorosamente o ciclo **TDD (Red → Green → Refactor)**. O desenvolvimento será feito de *dentro para fora*:

1. **Testes de Unidade (Domain - VOs e Entidades):** 
   - *Mentalidade TDD:* Escrever o teste de uma `LeituraUmidade` com valor `-10%` **ANTES** de criar a classe. O teste falhará (*Red*).
   - *Implementação:* Criar a classe e a lógica para lançar `DomainValidationException` (*Green*).
   - *Objetivo:* Garantir que a fundação de domínio é inviolável.

2. **Testes de Unidade (Domain Service):** 
   - *Mentalidade TDD:* Escrever o teste do `ValidadorNecessidadeIrrigacao` enviando solo em 5% (Crítico) e chuva de 30mm. Validar se retorna `false`.
   - *Objetivo:* Isolar e testar exaustivamente a inteligência algorítmica principal (Bloqueio Climático) sem depender de infraestrutura.

3. **Testes de Aplicação (Use Case):** 
   - *Mentalidade TDD:* Testar o fluxo completo orquestrado pelo `AvaliarIrrigacaoUseCase` injetando repositórios e gateways "Fakes" (em memória).
   - *Objetivo:* Validar, sem tocar no banco de dados real, se o UseCase processa timeouts corretamente e persiste o status `Em_Falha`.
   - **Rastreabilidade (SKILL):** Teste de aceitação validando a execução do **RF03** (Acionar Válvula) e do **RF04** (Registrar Falha).

4. **Testes de Integração (Adapters / Infra):** 
   - *Mentalidade TDD:* Por último, testar a implementação real de banco `IrrigacaoRepositoryPostgres` contra um banco de dados isolado (ex: Testcontainers).
   - *Objetivo:* Homologar estritamente o ORM e as Queries de persistência (mapeamento do DER), validando as bordas sujas do sistema.
   - **Rastreabilidade (SKILL):** Teste garantindo o armazenamento perene exigido pelos **RF01** e **RF04**.
