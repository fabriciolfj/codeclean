# Detalhes do garbage collection
- hotspot java, utiliza várias técnicas para melhorar a performance do gc, como: evitar fragmentação, localidade/refrência para busca dos dados e compactuação.
- alguns comandos que podemos utilizar.
```
# Habilitar principais otimizações
java -XX:+UseCompressedOops 
     -XX:+UseCompressedClassPointers 
     -XX:+UseStringDeduplication
```
- outro ponto, o tempo de recorrência do gc, não há um cronograma, ele e executado conforme a necessidade de memória, ou seja, com base em eventos.
## STW
```
O Stop-the-World (STW) no Garbage Collector (GC) do Java é um mecanismo importante para garantir a consistência durante a coleta de lixo. Vou explicar como isso funciona:

Mecanismo do STW:
O GC precisa pausar todas as threads da aplicação (application threads) que estão manipulando objetos
Durante o STW, apenas as threads do GC continuam executando
Isso é necessário para evitar que objetos sejam modificados durante a coleta

Como o GC consegue fazer o STW:
Usa um mecanismo chamado "safepoint"
Cada thread da JVM periodicamente checa por requisições de safepoint
Essas checagens são inseridas em pontos específicos do código durante a compilação JIT
Locais comuns para safepoints:

Final de loops
Chamadas de método
Alocação de objetos

Processo:
O GC sinaliza uma requisição de safepoint
As threads continuam executando até atingirem um ponto de safepoint
Quando atingem o safepoint, as threads são suspensas
O GC aguarda todas as threads atingirem um safepoint
Só então inicia a coleta de lixo
Após terminar, as threads são liberadas para continuar

Otimizações modernas:
GCs mais recentes como ZGC e Shenandoah minimizam o STW
Usam técnicas como coleta concorrente
Mantêm STW apenas para operações críticas e muito breves
Mesmo assim, ainda precisam do mecanismo de safepoint

O STW é fundamental para garantir a consistência da heap durante a coleta de lixo, mesmo que seja um dos principais causadores de latência em aplicações Java.
```
## safepoints
```
Os safepoints são pontos específicos no código Java onde é seguro para a JVM pausar a execução das threads. Vou explicar em detalhes:

Definição Técnica:
São locais no código onde o estado da thread é totalmente conhecido pela JVM
Todas as referências de objetos estão em locais conhecidos
O mapeamento entre registradores e variáveis está bem definido
A pilha de execução está em um estado consistente

Onde são inseridos:
Fim de loops
Antes/depois de chamadas de método
Retorno de métodos
Final de blocos de código
Durante alocações de objetos
Em operações de locking/synchronization

Como funcionam:
javaCopy// Exemplo conceitual de como safepoints funcionam
while(condicao) {  // Safepoint inserido no teste do loop
    operacao();    // Safepoint antes da chamada do método
    if(x > 0) {    
        return;    // Safepoint antes do return
    }
}              // Safepoint no final do loop

Por que são necessários:
Permitem que o GC saiba exatamente onde estão todas as referências
Facilitam a pausa sincronizada de threads (STW)
São essenciais para deoptimização de código JIT
Permitem profile sampling preciso
Facilitam o debug da JVM

Comportamento:
A JVM pode requisitar que threads parem em safepoints
Threads checam periodicamente se há requisições de safepoint
Se há uma requisição, a thread para quando atinge o próximo safepoint
A thread só continua quando a requisição de safepoint é liberada

Impacto no desempenho:
Safepoints adicionam overhead mínimo em execução normal
O compilador JIT otimiza as checagens de safepoint
Em loops muito pequenos, podem ser removidos para melhor performance
Loops muito grandes podem ter safepoints adicionais inseridos

Debug e Monitoramento:
bashCopy# Exemplo de comando para ver estatísticas de safepoint
java -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1

Casos especiais:
Código nativo (JNI) não tem safepoints
Algumas operações críticas podem ser marcadas como "no safepoint"
Threads em código nativo precisam retornar ao Java para checar safepoints

É importante notar que safepoints são uma implementação interna da JVM e os desenvolvedores geralmente não precisam se preocupar diretamente com eles, mas entender seu funcionamento ajuda a compreender melhor o comportamento da JVM e do GC.
```

## Tri-Color Marking
```
O Tri-Color Marking é um algoritmo importante usado em Garbage Collection, especialmente em coletores concorrentes. Vou explicar seu funcionamento:

As três cores:
Branco: Objetos potencialmente não alcançáveis (possível lixo)
Cinza: Objetos alcançáveis mas ainda não escaneados
Preto: Objetos alcançáveis e já completamente escaneados

Invariantes do algoritmo:
Forte: Nenhum objeto preto aponta diretamente para um branco
Fraco: Todo objeto branco alcançável por um preto é também alcançável por um cinza

Fases principais:
Inicialização: Todos objetos começam brancos
Marcação inicial: GC roots marcados como cinza
Marcação: Objetos cinzas são processados e suas referências marcadas
Finalização: Objetos brancos são coletados

Desafios em ambientes concorrentes:
Mutador (aplicação) pode modificar referências durante marcação
Precisa manter invariantes mesmo com modificações concurrent
Usa barreiras de escrita/leitura para consistência

Vantagens:
Permite coleta incremental e concorrente
Facilita visualização do progresso do GC
Reduz pause times em GCs modernos

Implementações na prática:
G1 GC usa variação do tri-color
ZGC e Shenandoah usam conceitos similares
CMS também baseia-se neste conceito

Otimizações comuns:
SATB (Snapshot At The Beginning)
Refinamento incremental de cards
Barreiras otimizadas por JIT

Este algoritmo é fundamental para GCs modernos e permite coleta de lixo com baixa latência em aplicações Java.
```

## Forwarding Pointers 
```
Os Forwarding Pointers são um mecanismo importante usado durante a compactação de memória no GC. Vou explicar em detalhes:

Conceito básico:
São ponteiros temporários usados durante movimentação de objetos
Permitem atualizar referências para objetos que foram movidos
Mantêm um link entre localização antiga e nova do objeto

Processo em 3 fases:
Fase 1: Marcar objetos vivos e calcular novos endereços
Fase 2: Instalar forwarding pointers nos objetos
Fase 3: Atualizar todas as referências usando os forwarding pointers

Uso prático:
Durante compactação de heap
Em coletores que movem objetos (como G1)
Para manter consistência durante relocação

Benefícios:
Permite movimentação segura de objetos
Facilita atualização de referências
Suporta compactação incremental
Reduz fragmentação de memória

Desafios:
Overhead de memória temporário
Necessidade de sincronização em ambientes paralelos
Complexidade na implementação

Otimizações comuns:
Reutilização de bits no header do objeto
Processamento paralelo de atualizações
Técnicas de compactação sliding

Impacto no desempenho:
Pause time durante instalação dos ponteiros
Overhead de memória durante a compactação
Benefício de redução de fragmentação

Os forwarding pointers são essenciais para implementar GCs que movem objetos e são uma parte fundamental de coletores modernos como G1 e Shenandoah.
```
## G1
```
G1 (Garbage First) é um coletor de lixo moderno da JVM. Vou explicar seus principais aspectos:

Estrutura da Heap:
Dividida em regiões (regions) de tamanho fixo
Cada região pode ser Eden, Survivor, Old ou Humongous
Tamanho padrão da região é calculado baseado no heap (1-32MB)
Regiões Humongous são para objetos grandes (>50% do tamanho da região)

Características principais:
Coleta incremental e paralela
Prioriza regiões com mais garbage ("Garbage First")
Predictable pause times
Compactação automática durante a coleta
SATB (Snapshot At The Beginning) para marcação concorrente

Fases de coleta:
Initial Mark (STW)
Root Region Scan (Concurrent)
Concurrent Mark
Remark (STW)
Cleanup (STW + Concurrent)
Copying (STW)

Remembered Sets (RSet):
Cada região mantém track de referências externas
Permite coleta independente de regiões
Otimiza scanning durante coleta

Collection Sets (CSet):
Conjunto de regiões a serem coletadas
Escolhidas baseadas em efficiency
Balanceia tempo de pausa vs espaço recuperado

Tuning comum:
bashCopy# Exemplos de parâmetros importantes
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:G1HeapRegionSize=16m
-XX:InitiatingHeapOccupancyPercent=45

Vantagens:
Pause times previsíveis (não há garantia que será acatado)
Bom throughput
Fragmentação reduzida
Melhor escalabilidade em heaps grandes

Desvantagens:
Overhead de memória (RSets)
Complexidade de tuning
Pode ter pause times maiores que CMS em alguns casos

Situações ideais de uso:
Heaps grandes (>4GB)
Requisitos de latência moderados
Aplicações com muitos threads
Sistemas com múltiplos cores

O G1 é o GC default desde Java 9 e é uma excelente escolha para muitas aplicações modernas, especialmente aquelas que precisam de pause times previsíveis em heaps grandes.
```
### g1 old
```
O G1 Old Generation (G1Old) é parte do G1 Garbage Collector, focando especificamente na gerração Old. Vou explicar seus principais aspectos:

Características Principais:


Parte do G1GC (Garbage First)
Gerencia objetos de vida longa
Usa regiões de tamanho fixo
Coleta incremental


Funcionamento:


Objetos sobreviventes da Young Gen são promovidos
Usa RSet (Remembered Sets) para rastreamento
Collection Set (CSet) determina regiões a serem coletadas
Evacuação de regiões selecionadas


Mixed Collections:


Coleta tanto regiões Old quanto Young
Acontece quando Old Gen está muito cheia
Tenta manter dentro dos objetivos de pausa
Seleciona regiões com mais espaço livre primeiro


Configurações Importantes:

bashCopy-XX:G1MixedGCLiveThresholdPercent=85  # Limite para mixed collections
-XX:G1HeapWastePercent=5              # Desperdício tolerável
-XX:G1MixedGCCountTarget=8            # Número alvo de mixed GCs

Situações de Full GC:


Quando não consegue recuperar memória rápido o suficiente
Se a heap ficar muito fragmentada
Quando marking não termina a tempo
Em caso de falha na evacuação


Otimizações:


Coleta regiões com mais garbage primeiro
Balanceia tempo de pausa vs throughput
Usa SATB (Snapshot-At-The-Beginning)
Marking concorrente

O G1Old é crucial para o funcionamento eficiente do G1GC, especialmente em aplicações com grandes heaps e necessidade de baixa latência.
```

### gc flutuante
- quando objetos mortos sao mantigos vivos por causa de referência de outros objetos mortos.
### g1 coleta mista
- thread do gc são executadas simultaneamente com as threads do aplicativo.
- throughput da app e reduzido nesse momento.

### rsets
```
O Reset (R-Set ou Remembered Set) é uma estrutura de dados importante usada no gerenciamento de memória da JVM, especialmente relacionada ao Garbage Collection.
Principais aspectos dos RSets:

Função Principal:


Mantém registro de referências entre diferentes gerações de objetos
Rastreia quais objetos na geração mais velha apontam para objetos na geração mais nova
Evita a necessidade de varrer toda a heap durante coletas de lixo menores


Como Funciona:


Divide a heap em "cards" (pequenas regiões de memória)
Marca cards que contêm referências para objetos em gerações mais novas
Quando um objeto é atualizado, o card correspondente é marcado como "dirty"


Benefícios:


Melhora a performance do GC
Reduz a pausa do GC
Permite coletas parciais eficientes
Otimiza o tempo de varredura da memória


Uso Prático:


Fundamental para GC geracional
Importante em collectors como G1 e CMS
Ajuda a manter a eficiência em aplicações com grandes heaps


Custo:


Consome memória adicional
Requer processamento para manutenção
Pequeno overhead durante a execução do programa

O RSet é uma otimização crucial que permite que o Garbage Collector trabalhe de forma mais eficiente, focando apenas nas áreas relevantes da memória em vez de ter que examinar toda a heap.
```

## ihop
```
O iHOP (Inline Hood Optimization Program) é uma técnica de otimização da JVM HotSpot que faz parte do JIT (Just-In-Time) Compiler.
Principais características:

Funcionamento Básico:


Otimiza chamadas de método inline
Evita overhead de chamadas de método
Melhora o desempenho do código em tempo de execução


Otimizações:


Inlining de métodos pequenos
Eliminação de dispatch virtual
Redução de overhead de chamadas
Melhora o cache locality


Benefícios:


Melhor performance
Redução do uso de memória
Menos chamadas de método
Código mais eficiente


Quando é usado:


Em métodos chamados frequentemente
Em métodos pequenos
Em hot spots da aplicação
Durante otimizações JIT


Limitações:


Não funciona com todos os tipos de métodos
Pode aumentar o tamanho do código
Tem limites de profundidade de inlining
Requer análise em tempo de execução

O iHOP é uma das muitas otimizações que a JVM faz automaticamente para melhorar o desempenho das aplicações Java.
```

## alocação
- quantidade de memoria utilizada pelo objeto

## tempo de via do objeto
- muito dificil de medir ou estimar
- o tempo de vida (lifecycle) de um objeto é o período entre sua criação (instanciação) e o momento em que ele se torna elegível para coleta pelo Garbage Collector, ou seja, quando não há mais referências alcançáveis a ele.
## pico de alocacao
```
Causas comuns:


Processamento em lote de muitos dados de uma vez
Cache sendo preenchido ou recarregado
Upload de arquivos grandes
Operações de bulk insert no banco de dados
Geração de relatórios grandes
Respostas HTTP com muito conteúdo


Impactos:


Pode disparar GCs mais frequentes
Maior pressão na Young Generation
Pode causar promoção prematura para Old Gen
Risco de OutOfMemoryError se muito intenso
Aumento no tempo de pausa do GC
Degradação de performance da aplicação


Como identificar:


Monitoramento de métricas de alocação
Análise de GC logs
Ferramentas como JVisualVM ou JFR
Aumento repentino no uso de memória
GCs mais frequentes que o normal


Soluções comuns:


Implementar processamento em lotes menores
Usar paginação
Controlar tamanho máximo de uploads/downloads
Object pooling para objetos grandes
Otimizar algoritmos que geram muitas alocações
Ajustar tamanho das generations do GC
```
## stw
```
O STW (Stop-The-World) é um momento durante a execução do Garbage Collector onde todas as threads da aplicação são pausadas para que o GC possa executar seu trabalho.
Detalhando o funcionamento:

O que acontece durante STW:


Todas as threads da aplicação são pausadas
Nenhum novo objeto pode ser alocado
Nenhuma referência pode ser atualizada
Apenas as threads do GC continuam executando


Quando ocorre:


Durante operações críticas do GC que precisam de consistência
Na fase de marcação de objetos vivos
Durante compactação/movimentação de objetos
Quando precisa atualizar referências


Impactos:


Causa latência na aplicação
Pode ser problemático para aplicações que precisam de resposta rápida
O tempo de pausa varia conforme tamanho do heap e quantidade de objetos


Em diferentes coletores:


ParallelOld: STW durante todo o processo de coleta
CMS: Tenta minimizar STW fazendo parte do trabalho concorrentemente
G1: Usa STW mais curtos mas frequentes
ZGC: Tenta manter STW abaixo de 10ms


Monitoramento:


GC logs mostram duração dos STW
Pode ser monitorado via JMX
Ferramentas como JVisualVM mostram pausas
```

## EDEN
- aonde os objetos novos ficam

## TLABS
- thread local allocation buffers
- é uma técnica de otimização importante utilizada pela JVM para melhorar a performance de alocação de objetos. Cada thread recebe seu próprio buffer de memória para alocação de objetos, o que reduz a contenção entre threads durante a alocação.
- quando uma thread precisa alocar um novo objeto, a JVM utiliza o TLAB (Thread Local Allocation Buffer) que é uma área específica da heap reservada para cada thread.
- O processo funciona assim:
  - Cada thread recebe seu próprio TLAB na Young Generation (Eden Space)
  - Quando a thread cria um novo objeto, ele é inicialmente alocado no TLAB dessa thread
  - Isso evita sincronização entre threads durante a alocação (melhor performance)
- Quando o TLAB de uma thread fica cheio:
  - A thread recebe um novo TLAB
  - O TLAB antigo se torna parte normal do Eden Space
  - Se não houver espaço para um novo TLAB, ocorre uma coleta de lixo (GC)
- A alocação no TLAB é mais rápida porque:
  - Não precisa de sincronização entre threads
  - É uma operação simples de ponteiro
  - Reduz contenção na heap compartilhada 
- e outro ponto, quando thread recebe um objeto, ela aponta para outro endereço vazio na memória.

## exemplo simplificado como funciona o gc
```
Considere uma aplicação processando uma lista de pedidos:

Estado Inicial da Heap:

CopyEden (vazio)
S0 (vazio)
S1 (vazio)
Old Gen (alguns objetos antigos)

Aplicação começa a criar objetos:

CopyEden:
- Pedido#1
- Pedido#2
- Pedido#3
- Itens do Pedido
- Objetos temporários

Eden fica cheio, dispara Minor GC:


JVM marca objetos vivos no Eden
Pedidos #1, #2, #3 ainda têm referências (estão vivos)
Objetos temporários não têm referências (são lixo)
Objetos vivos são movidos para S0
Eden é limpo

CopyEden (vazio)
S0:
  - Pedido#1 (age=1)
  - Pedido#2 (age=1)
  - Pedido#3 (age=1)
S1 (vazio)

Mais alocações acontecem:

CopyEden:
  - Pedido#4
  - Pedido#5
  - Mais objetos temporários
S0:
  - Pedidos anteriores
S1 (vazio)

Novo Minor GC:


Objetos vivos do Eden vão para S1
Objetos sobreviventes de S0 também vão para S1
Age dos objetos é incrementado

CopyEden (vazio)
S0 (vazio)
S1:
  - Pedido#1 (age=2)
  - Pedido#2 (age=2)
  - Pedido#3 (age=2)
  - Pedido#4 (age=1)
  - Pedido#5 (age=1)

Se objetos sobrevivem a várias coletas (threshold, geralmente 15):


São promovidos para Old Generation
Exemplo: após mais GCs, Pedidos #1, #2, #3 vão para Old Gen

CopyEden: (novos objetos)
S0/S1: (objetos mais novos)
Old Gen:
  - Pedido#1
  - Pedido#2
  - Pedido#3
Pontos importantes:

Minor GC: coleta apenas Young Gen (Eden + Survivors)
Major GC: coleta Old Gen
Full GC: coleta toda a heap
Survivor spaces (S0 e S1) nunca são usados ao mesmo tempo
Objetos grandes podem ir direto para Old Gen
Promoção prematura pode ocorrer se Survivor ficar cheio

Este é um exemplo simplificado, mas mostra o ciclo básico de vida dos objetos na heap. 
```

# Event sourcing DDD
```
Domain Events:

Representam mudanças significativas no domínio
Acrescenta uma linha de tempo nos dados
São imutáveis e contêm toda informação necessária
Mantêm o histórico completo do agregado.
Não salvamos o agregado, por exemplo Account, apenas os eventos e o recuperamos eles com base no tempo (o mais recente, o último).

Aggregate Root:
BankAccount é o agregado que encapsula as regras de negócio
Gera eventos para cada mudança de estado
Mantém sua consistência através de invariantes

Event Store:
Armazena todos os eventos do sistema
Garante a ordem e consistência dos eventos
Permite reconstruir o estado do agregado

Repository:
Salva e recupera agregados usando o Event Store
Gerencia a concorrência através de versionamento
Reconstrói agregados a partir do histórico de eventos


Benefícios desta abordagem:
Auditoria:
Histórico completo de mudanças
Rastreabilidade de todas as operações
Possibilidade de debugging temporal

Evolução do Sistema:
Facilita mudanças no modelo de domínio
Permite reconstruir estados passados
Possibilita novas visões dos dados

Escalabilidade:
Eventos podem ser processados assincronamente
Facilita CQRS (Command Query Responsibility Segregation)
Permite otimizações de performance


Desafios e Considerações:
Complexidade:
Requer mais código inicial
Curva de aprendizado maior
Necessidade de lidar com versioning de eventos

Performance:
Reconstrução de estado pode ser custosa
Necessidade de snapshots para otimização
Gerenciamento de concorrência mais complexo

Modelagem:
Eventos devem ser bem definidos
Necessidade de versionamento de eventos
Cuidado com o tamanho dos agregados
```
- Exemplo
```
public interface EventStore {
    void saveEvents(UUID aggregateId, List<DomainEvent> events, long expectedVersion);
    List<DomainEvent> getEvents(UUID aggregateId);
}

// Implementação com JPA
@Entity
@Table(name = "event_store")
public class EventEntry {
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(name = "aggregate_id")
    private UUID aggregateId;
    
    @Column(name = "event_type")
    private String eventType;
    
    @Column(name = "event_data", columnDefinition = "jsonb")
    private String eventData;
    
    @Column(name = "version")
    private long version;
    
    @Column(name = "timestamp")
    private LocalDateTime timestamp;
}

@Repository
public class JpaEventStore implements EventStore {
    private final EntityManager entityManager;
    private final ObjectMapper objectMapper;
    
    public JpaEventStore(EntityManager entityManager, ObjectMapper objectMapper) {
        this.entityManager = entityManager;
        this.objectMapper = objectMapper;
    }
    
    @Override
    @Transactional
    public void saveEvents(UUID aggregateId, List<DomainEvent> events, long expectedVersion) {
        // Verificar versão atual
        Query query = entityManager.createQuery(
            "SELECT MAX(e.version) FROM EventEntry e WHERE e.aggregateId = :aggregateId"
        );
        query.setParameter("aggregateId", aggregateId);
        Long currentVersion = (Long) query.getSingleResult();
        
        if (currentVersion != null && currentVersion != expectedVersion) {
            throw new ConcurrencyException(
                String.format("Concurrent modification on aggregate %s", aggregateId)
            );
        }
        
        // Salvar novos eventos
        for (DomainEvent event : events) {
            EventEntry entry = new EventEntry();
            entry.setAggregateId(aggregateId);
            entry.setEventType(event.getClass().getName());
            entry.setVersion(event.getVersion());
            entry.setTimestamp(event.getTimestamp());
            
            try {
                entry.setEventData(objectMapper.writeValueAsString(event));
            } catch (JsonProcessingException e) {
                throw new RuntimeException("Error serializing event", e);
            }
            
            entityManager.persist(entry);
        }
    }
    
    @Override
    public List<DomainEvent> getEvents(UUID aggregateId) {
        TypedQuery<EventEntry> query = entityManager.createQuery(
            "SELECT e FROM EventEntry e WHERE e.aggregateId = :aggregateId ORDER BY e.version",
            EventEntry.class
        );
        query.setParameter("aggregateId", aggregateId);
        
        return query.getResultList().stream()
            .map(this::deserializeEvent)
            .collect(Collectors.toList());
    }
    
    private DomainEvent deserializeEvent(EventEntry entry) {
        try {
            Class<?> eventClass = Class.forName(entry.getEventType());
            return (DomainEvent) objectMapper.readValue(entry.getEventData(), eventClass);
        } catch (Exception e) {
            throw new RuntimeException("Error deserializing event", e);
        }
    }
}

// Exemplo de uso com Spring Data JPA
@Repository
public class BankAccountRepository {
    private final EventStore eventStore;
    
    public BankAccountRepository(EventStore eventStore) {
        this.eventStore = eventStore;
    }
    
    public void save(BankAccount account) {
        List<DomainEvent> uncommittedChanges = account.getUncommittedChanges();
        if (!uncommittedChanges.isEmpty()) {
            eventStore.saveEvents(
                account.getId(),
                uncommittedChanges,
                account.getVersion() - uncommittedChanges.size()
            );
            account.markChangesAsCommitted();
        }
    }
    
    public BankAccount getById(UUID accountId) {
        List<DomainEvent> events = eventStore.getEvents(accountId);
        if (events.isEmpty()) {
            throw new EntityNotFoundException("Account not found: " + accountId);
        }
        
        BankAccount account = new BankAccount();
        account.loadFromHistory(events);
        return account;
    }
}

// Schema SQL
CREATE TABLE event_store (
    id BIGSERIAL PRIMARY KEY,
    aggregate_id UUID NOT NULL,
    event_type VARCHAR(255) NOT NULL,
    event_data JSONB NOT NULL,
    version BIGINT NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    CONSTRAINT uk_aggregate_version UNIQUE (aggregate_id, version)
);

CREATE INDEX idx_aggregate_id ON event_store (aggregate_id);
```

# OLTP vs OLAP
```
OLTP (Online Transaction Processing) e OLAP (Online Analytical Processing) são estilos de processamento em sistemas de gerenciamento de banco de dados.
Aqui estão algumas das principais características e usos de cada um.
  OLTP (Online Transaction Processing):
  OLTP é mais comum em sistemas de gerenciamento de transações diárias, onde a velocidade e eficiência são críticas. Por exemplo, sistemas bancários ou varejistas.
  Os sistemas OLTP gerenciam, manipulam e mantêm os dados transacionais (ou seja, detalhes de cada transação).
  OLTP envolve inserção, atualização e/ou exclusão de pequenas quantidades de dados.
  As transações OLTP são frequentemente curtas e rápidas.

OLAP (Online Analytical Processing):
  OLAP, por outro lado, é mais comum em sistemas de data warehouse, onde é crucial a análise detalhada dos dados.
  Os sistemas OLAP lidam com dados históricos que foram processados e agregados de forma a facilitar o processo de tomada de decisões, relatórios, previsões, análises de mercado, etc.
  OLAP envolve a manipulação de dados retirados de fontes de dados OLTP em uma escala muito grande.

As transações OLAP são geralmente complexas e podem levar horas ou mesmo dias para serem concluídas.
Essencialmente, OLTP é sobre o processamento eficiente das transações diárias, enquanto OLAP é sobre o processamento eficiente das consultas para análise de dados e relatórios. Ambos desempenham um papel crítico em diferentes aspectos da tomada de decisões das empresas.

```

# Pontos importantes DDD (domain driven design)
- linguagem onipreente
- contexto delimitado
- dominio e subdominio
 
## Diferença entre contexto delimitado e subdominio:
```
Contexto delimitado e subdomínio são dois conceitos relacionados a arquitetura de software, especificamente Domain-Driven Design (DDD):

**Contexto Delimitado**

- É uma delimitação clara dos processos de negócio, regras e responsabilidades que pertencem AQUELE contexto.

- É uma fronteira ao redor de um modelo de domínio para isolá-lo das complexidades externas.

- Exemplos: contexto de pedidos, contexto de entregas, contexto de pagamentos.

**Subdomínio**

- Representa uma área de conhecimento ou atividade dentro de um negócio que é única e pode ser isolada logicamente do restante.

- Um contexto delimitado pode conter vários subdomínios, cada um com sua própria linguagem e regras.

- Exemplos: dentro do contexto de pedidos, pode haver o subdomínio de crédito, subdomínio de precificação, subdomínio de inventário.

Em resumo:

- Contexto delimitado é um isolamento vertical de responsabilidade.

- Subdomínio é uma divisão horizontal dentro de um contexto, para agrupar especialidades.

O contexto delimitado define uma fronteira, dentro da qual podemos modelar subdomínios quando necessário.
```

# dicas na criação de uma app backend
- crie annotations customizadas (vinculando a constraint validation), para uso em validações
- em caso de api rest, coloque o openapi
- delegue as mensagens de validações para um arquivo externo, como message.properties ou exceptions.properties
- use arquitetura clean, ou seja, separe o dominio da app dos frameworks,
- caso utilize um container context, como spring, crie uma annotation personalizada como @usecase e esta vincule a @component(spring) ou @applicationcontext (quarkus)
- use contratos de fronteira para o domain, ou externalize o domain para adpater (validar caso a caso)
- caso precise validar os dados de uma mensagem, faça o dto implementar constraint validation e executar a validação.

# Gerenciamento de memória java

## configurando um app para gerar um headdump
- quando o app ficar sem memória ele gerará um arquivo headpdump.bin na raiz
```
-XX:+HeapDumpOnOutOfMemoryError    
-XX:HeapDumpPath=heapdump.bin
```
- para gerar via comamnd line
```
jps -l para achar o pid
jmap -dump:format=b,file=C:/DA/heapdump.bin 25320
```
- podemos importar o dump em uma ferramenta, como o visualvm, para investigar
- ao importar podemos ver o resumo do que está ocorrendo, objetos que estão dentro do heap ou até efetuar consultas sql
- no exemplo abaixo para identificar o número de referências, ou seja, quando tenho um alto número de instâncias e um pequeno número de referência, é sinal de vazamento de memória
```
select { product: p.name, count: count(referrers(p))} from model.Product p
```

## stack
- local na memoria heap, utilizada pela thread
- aonde ficam armazenadas as variáveis locais e referência a instância de objeto
- a stack usa o conceito de LIFO (ultimo que entrada, primeiro que sai)
- a camada é removida, quando o bloco de codigo termina (quando termina sua execução, lança uma exception ou tem um return), exemplo

```
public static void main(String [] args) { main cria uma camada na pilha
  int x = 10;
  a(); cria uma camada na filha e dp que chamou esse metodo, esta camada e removida
  b(); 
}
 
public static void a() {
  int y = 20;
}
 
public static void b() {
  int y = 30;
}
// nesse exemplo quando termina o metodo main, a thread encerra sua execução, a pilha fica vazia e é removida
// por fim a thread entra no ciclo de encerramento
```

## head
- local aonde armazena-se as instâncias dos objetos
- compartilhado entre as threads
- problemas no head, como head cheia (outofmemoryError) são mais difíceis de resolver, pois quem indica o problema e a thread que não conseguiu armazenar e não a que encheu.

## metaespaço
- local na memória na jvm para armazear os tipos de dados usados para criar instâncias

# troubleshooting-java
- para investigar problemas em app, devemos utilizar ferramentas criadoras de perfis
  - por exemplo temos o visualvm, para monitoramente e análise de desempenho para app java.
  - caso tenha problemas como codigo 62, utiliza como arg na vm do app -Xverify:none
- devemos criar perfis com base em uma amostragem e não da base de codigo por um todo.

## problema nas consultas n+1
- Quando uma estrutura precisa obter dados de várias tabelas, ela geralmente sabe compor uma consulta e obter todos os dados em uma única chamada. No entanto, se você não usar o framework corretamente, ele pode pegar apenas parte dos dados com uma consulta inicial e então, para cada registro inicialmente recuperado, executar uma consulta separada. Assim, ao invés de executar apenas uma consulta, o framework enviará uma consulta inicial mais N outras (uma para cada um dos N registros recuperados pela primeira); chamamos isso de problema de consulta N+1, que geralmente cria uma latência significativa executando muitas consultas em vez de apenas uma.  

## Vazamento de memória
- posíveis causas:
  - adicionando instâncias na memória, mas nunca as remove
  - quando estamos referenciando objetos que não são mais usados
  - threads abertas que não terminam seu processo
  - GC não gasta nenhum recurso de CPU. Isso também não é um bom sinal. Em outras palavras, o aplicativo gasta muito poder de processamento, mas não processa nada. Esses sinais geralmente indicam threads zumbis, que geralmente são consequência de problemas de simultaneidade.
  - quando ocorre um outofmemory error, precisamos de um head dump para verificar a causa
 
## desempenho
- verifique o tempo de execução e o tempo de cpu, caso aja tempo de execução e o tempo de cpu seja 0, o app está esperando por algum

## dump threads
- ideal para analisar congelamento do app (deadloack de threads)
- para pegar o pid do processo jps -l e jstack <<PID>> > test.dump para gerar o dump
- podemos importar o dump em https://fastthread.io/ para facilitar a leitura ou ler o arquivo bruto
```
topico                 priodidade no so, tempo de cpu, tempo total                   id da thread no so descricao do estado da thread  identificacao                
"_Producer" #16 prio=5 os_prio=0 cpu=46.88ms elapsed=763.96s tid=0x000002f964987690 nid=0xcac waiting for monitor entry  [0x000000fe5ceff000]
   java.lang.Thread.State: BLOCKED (on object monitor) //trace
    at main.Producer.run(Unknown Source)
    - waiting to lock <0x000000052e0313f8> -- esta dando lock no recurso com essa identificacao identificacao
 (a java.util.ArrayList)     
    - locked <0x000000052e049d38> 
 (a java.util.ArrayList)     
 
"_Consumer" #18 prio=5 os_prio=0 cpu=0.00ms 
 elapsed=763.96s tid=0x000002f96498b030 
 nid=0x4254 waiting for monitor entry  [0x000000fe5cfff000]
   java.lang.Thread.State: BLOCKED (on object monitor)
    at main.Consumer.run(Unknown Source)
    - waiting to lock <0x000000052e049d38> (a java.util.ArrayList)
    - locked <0x000000052e0313f8> (a java.util.ArrayList)
```

## Soluções paliativas para o vazamento de memória  
- aumentar a memoria head: -Xmx1G por exemplo
- aumentar o metaspace (local aonde ficam os metadados das classes, metodos, campos da classe, nome da classe, pacote, modificadores de acesso e etc, necessários para a execução do app): -XX:MaxMetaspaceSize=100M
  - obs: situações aonde o metaspace é totalmente preenchido, é uma grande volume no uso de reflexão.
 
## JProfiler
- possui alguns recursos a mais que o visualVM, como:
  - grafico de chamadas
  - identificador se sua app está fechando corretamente as conexões com uma base relacional
  - gráfica de chamadas, demostra a pilha de chamadas 

## metodo esperando por si mesmo
- quando a sua thread foi travada

## thread bloqueada vs tread em espera
- thread e bloqueada em um bloco de codigo sincronizado
- thread em espera, o monitor definiu explicitamente para o estado bloqueado. Thread em espera pode continuar sua execução somente após o monitor informar explicitamente que ela pode prosseguir com sua execução.

# Patter saga
- O padrão Saga é um padrão de arquitetura de software usado para coordenar transações distribuídas em sistemas distribuídos e de longa duração. Ele oferece uma abordagem para lidar com a consistência de dados em operações complexas que envolvem múltiplas etapas ou serviços.

Em um sistema baseado no padrão Saga, uma transação é dividida em várias etapas (ou passos), cada uma representando uma ação a ser executada em um serviço específico. Cada etapa é encapsulada em uma unidade de trabalho atômica e possui a capacidade de desfazer (ou compensar) as operações realizadas, se necessário. Isso permite que o sistema reaja a falhas ou ações corretivas.

Existem dois modelos principais para a implementação do padrão Saga:

Modelo de Saga Baseado em Orquestração: Nesse modelo, existe um componente central chamado orquestrador que coordena todas as etapas da transação. O orquestrador decide a ordem das etapas, envia comandos para os serviços envolvidos e lida com possíveis erros e compensações. Ele controla todo o fluxo da transação.

Modelo de Saga Baseado em Coreografia: Nesse modelo, cada serviço envolvido na transação é responsável por coordenar suas próprias operações e interações com outros serviços. Cada serviço decide como reagir a eventos e solicitações recebidas e inicia suas próprias ações, enviando mensagens para outros serviços, conforme necessário. As interações entre os serviços formam uma "coreografia" que define o fluxo da transação.

Ambos os modelos têm suas vantagens e desvantagens, e a escolha entre eles depende do contexto e dos requisitos específicos do sistema.

O padrão Saga aborda alguns desafios comuns em transações distribuídas, como o problema do duas fases de confirmação (2PC) que pode causar bloqueios e latência em operações distribuídas. Em vez disso, o padrão Saga oferece uma abordagem mais flexível, permitindo que cada etapa seja projetada para ser atomicamente consistente e capaz de desfazer as alterações, se necessário.

No entanto, é importante considerar que a implementação do padrão Saga pode adicionar complexidade ao sistema, especialmente ao lidar com falhas, erros e compensações. É necessário ter um bom mecanismo de persistência de estado e registro de eventos para acompanhar o progresso da transação e garantir a consistência de dados.

Vale ressaltar que o padrão Saga não resolve todos os problemas de transações distribuídas, e sua implementação requer cuidado e consideração dos requisitos específicos do sistema.

# Dicas para um código limpo

- Cria-se métodos pequenos
- utilize sobrecarga para evitar parametros nulos
- valide os parametros de metodos publicos
- O nome dos métodos, deve refletir o que ele faz.
- O método deve validar, criar ou consultar, nunca mais de 1 processo junto.
- O método deve possuir  no máximo 3 parâmetros.
- O método deve fazer apenas uma coisa, uma abstração, um propósito.
- Não use parâmetros de saída.
- No código catch, apenas trate a exceção.
- Nunca duplique código.
- Objetos usam abstrações para esconder seus dados ou funções, estrutura de dados expõs seus dados e não possuem funções.
- Um módulo não deve enxergar o interior dos objetos que manipula.
- Evite cadeia de chamadas, tipo: get().get().
- Nunca retorne null.
- Nunca passe como parâmetro um Optional.
- Nunca passe null como parâmetro.
- Não tenha vários catchs.
- Ao usar libs de terceiros, encapsule em uma classe.
- Use o código que não existe, exemplo: caso precise de uma api, que não existe naquele momento, desenvolva uma interface que expôes métodos que você precise e implemente uma classe fake. Quando o código original estiver pronto, desenvolva um adapter.
- Não devemos enxergar como o código de terceiro funcione, no caso de dependências, e não mistrar o código deste com o nosso.
- Os testes devem ser claros
- Os testes devem pertencer a um domínio.
- Testes devem ser óbivos, sucintos.
- O idetal que cada teste tenha apenas um assert.
- As classes devem ser pequenas e ter 1 responsabilidade.
- As classes devem ter apenas um motivo para mudar.
- Classes devem possuir um pequeno número de variáveis de instância.
- Métodos que usam todas as variáveis de instância, são mais coesos.
- Quando tenho métodos que usam algumas variávies de instância, crie outra classe para os mesmos.
- Use o princípio do aberto e fechado.
- Um sistema evolui com expansão e não com alteração de código existente.
- Use o principio de inversão de dependência, ou seja, dependa de abstrações.
- Deixe o sistema mais desacoplado possível, ele será mais flexivel para testes e reutilização de código.
- Separe a criação de objetos da lógica do seu sistema. Use o pattern abstract factory.
- Separe a lógica de negócio de frameworks.
- Modular os domínios, ou seja, separe em pacotes.
- Não adote uma arquitetura invasiva, ou seja, que polua seu domínio com frameworks.
- No seu dominio, use pojos, ou seja, java puro.
- Efetue testes, o alto acoplamento dificulta os testes.
- Não duplique o código.
- Expresse o propósito no codigo.
- Coloque como sufixos os nomes dos padrões nas classes que os usam.
- Código voltado para concorrência, deve ficar separado do código não concorrente.
- Limite severamente o acesso a quaisquer dados que possam ser compartilhados, em um código concorrente.
- Faça cópia dos dados em uma aplicação concorrente.
- Em métodos assincronos, opte por variáveis locais.
- Não tenha mais de um método synchronized, dentro da mesma classe.
- Evite usar mais de um método em um objeto compartilhado.
- Não ignore falhas de sistemas, como se fossem isolados.
- Antes de adotar threads, certifique que seu código funciona sem elas.
- Evite chamar uma seção assincrona a outra sincrona.
- Se uma classe possui muitos tipos diferentes e métodos auxiliares, considere criar outra classe.
- Após concluir a codificação, limpe o código, não se contente vê-lo apenas funcionando.
- Forma mais sutil de duplicação de código, são algoritimos parecidos, adote o pattern template method para eliminar esse comportamento.
- Exilar corretamente as abstrações, ou seja: conceitos mais genéricos ou de alto nivel em classes bases, conceitos mais específicos em subclasses ou classes concretas. Organize o projeto, separando as classes bases das subclasses em pacotes diferentes.
- Devem-se declarar variáveis e funções, próximas aonde são usadas.
- Mantenha a consistência, ou seja, se adotar uma abordagem mantenha, exemplo: varaivel de retorno a uma chamada de api, chama-se responsePessoa, em outros pontos muda o sufixo mas mantenha-se o response, como responseProduto.
- Codigo morto, nunca utilizado, apague.
- Repense pontos onde métodos alteram dados, de outro objeto que ele está incluso. Exemplo:
```
public class CalculaHoras {

  public Valor calcular(Funcionario funcionario) {
    var horas = funcionario.getPonto().getHoras(); //não preciso saber do ponto exato que vem as horas
    var total = horas * 0.5; //esse método poderia estar dentro da classe funcionario.
  
  }

}
```
- Evite o uso de parâmetros seletores, ou seja, boolean e anums, pois eles selecionam comportamentos dentro da minha função.
- Evite classes statics, caso adote, garanta que no futuro não mude o comportamento da mesma para polifórmica.
- Torne dependência lógica em físicas, ou seja, se depende de outro objeto, não assume responsabilidade que pertença a tal.
- Os métodos devem fazer 1 coisa só.
- Encapsule condicionais, exemplo:
```
Prefira isso
if(isValido())

em vez disso
if (i >0 && i < 9) -- coloquei no método isValid

public boolean isValid() {
  return i > 0 && 0 < 9;
}
```
- Evite condicionais negativas.Exemplo 
```
if(!isValid())
```
- Deixe claro os acoplamentos temporários, exemplo: ordem de chamadas de funções para realizar algum procedimento.
- Separe os níveis de abstração.
- Escolha nomes no nível apropriado de abstração.
- Use uma nomenclatura padrão aonde for possível.
- O nome da função deve descrever o que ela realmente faz, por exemplo:
```
public Cliente getCliente() {
  var cliente = findyFirst();
  
  if (cliente == null) 
    return new Cliente()
} esse método faz mais do que buscar o primeiro cliente, se não existir ele cria. O nome correto seria returnFirstOrCreateCliente
```

### Josua
- Em vez de retornar uma lista nula, retorne uma lista vazia.
- Valide os parâmetros logo na entrada.
- Métodos com mais de 3 parâmetros, considere passar um tipo que os represente.
- Em vez de passar um boolean, considere um enum como parâmetro.
- Declare com final as variáveis e parâmetros.
- De preferência a referência de métodos, em vez de lambda.
- Use as interfaces funcionais padrões.
- Cuidado ao uso de stream parallel.
- Não use sobrecarga com os mesmos números de parâmetros.
- Cuidado no uso de sequência de parâmetros de mesmo tipo.
- De preferência a composição em vez da herança.
- Quando precisar fechar alguma conexão, uso o try-with-resources.
- Use interface para definir tipos.
- De preferência a interfaces em vez de classes abstratas.
- Caso você não garante o retorno do objeto, use o return Optional.empty().
- Não retorne um primitivo optional, uso por exemplo OptionalInt.
- Não use optionals com maps, chaves ou indices de arrays.
- Cuidado no uso do optional, por questão de desempenho.
- Declare a variável local onde a mesma será utilizada.
- Todas as variáveis locais devem ser inicializadas.
- De preferência ao foreach em vez do for tradicional.
- Use o bigdeciaml para valores monetarios.
- De preferência a tipos primitivos em vez de empacotados.

### Arquitetura clean
- Orientação a objeto: é a habilidade de obter controle absoluto, através do uso do polimorfismo, sobre cada depência de código fonte do sistema.

#### SOLID
 - SRP : um módulo deve ter uma, e apenas uma razão para mudar (ou pertencer a um ator).
 - OCP (aberto para mudança, fechado para modificação) : objetivo do sistema ser facil de extender, sem que a mudança gere alto impacto. Para concretizar esse objetivo, particionamos o sistema em compoenentes e organizamos esses componentes em uma hierarquia de dependência que proteja os camponentes de nivel mais alto das mudanças em componentes de nivel mais baixo: Por exemplo: quero proteger o compoente A do componentes B, componente B deve depender do componente A para isso.
 - LSP (substituição de liskov) : capacidade de substituir as entidades filhas pelas pais, para usamos interfaces e heranças (polimorfismo).
 - ISP (segregação de interfaces) : é prejudicial depender de módulos que contenham mais elementos que voce precisa, dependa apenas daquilo que você necessita e separe-as em interfaces.
 - DIP (inversão de dependência): dependa de abstrações e não de implementações, utiliza-se o pattern obstract factory.

#### Diferença entre regras de dominio e regras da aplicação
- Regras de dominio, são regras que dificilmente mudam, e devem existir sem a aplicação (como por exemplo, calcular juros sobre um emprestimo).
- Regras da aplicação, encontradas no caso de uso, são regras que automatizam o dominio, e podem mudar com mais frequencia (forma de buscar contato de quem solicitou emprestimo).

#### Componentes
- componente estável: quando ele nao depende de ninguem e outros componentes depende dele (independente e responsável)
- componente instável: quando ele depende de outros, dependencia acima do numero dos que depende dele), dependente e irresponsável.
- Evite componente estável depender de componente instável, caso necessite, use DIP (inversão de dependencias).
- Utilize métricas: nivel de abstração e nivel de estabilidade do seu componente.

#### Arquitetura
- Para um bom arquiteto, deve deixar os detalhes mais abertos possíveis, pois eles não importam.
- Todos os sistemas de software podem ser decompostos em dois elementos principais: politica e detalhes.
- Política: engloba todas as regras e procedimentos de negócios, onde está o verdadeiro valor do sistema.
- Detalhes: itens necessários, como servidores, banco de dados, protocolo de comunicação e etc.
- Objetivo do arquiteto é criar uma forma para o sistema que reconheça a política como elemento essencial e torne os detalhes irrelevantes para essa política.
- Uma boa arquitetura, deve demonstrar qual negócio o sistema gerencia, por exemplo: sistema de rh, sistema de farmácia e etc, e não quais frameworks utiliza.
- Uma boa arquitetura deve suportar:
  - casos de uso e operação do sistema
    - deixe claro no sistema os pontos mais importantes
  - manutenção do sistema
    - que não seja custosa e agilidade na detecção dos problemas
  - operação:
    -  que facilita a escalabilidade da aplicação por exemplo.
  - desenvolvimento do sistema
    - facilite a evolução do sistema
    - uma escrutura que facilite diversas equipes trabalhar
  - implantação do sistema.
    - facilitar a implantação do sistema
    - após o build da app, ela ja é implantada, sem necessidade de scripts ou diretórios executados manualmente.
  - Deixe as opções abertas
  - Desacoplando camadas
    - Separar camadas que mudam por propósitos diferentes, exemplo: validação de entrada da aplicação, pertence a app, regras de juros pertence ao domínio.
  - Desacoplando casos de uso
    - Casos de usos diferentes, mudam em um ritmo e por razões diferentes.
  - Duplicação real vs duplicação acidental: real, constatado código duplicado no mesmo serviço. acidental: código parecidos em serviços diferentes(não unifique).
    
- Sugestão separação por casos de uso:
  - criarpedido
    - repositorio
    - service 
    - domain
    - ui
  - apagarpedido
    - repositorio
    - service
    - domain
    - ui

- arquietura de um sistema é definida pelos limites específicos dentro desse sistema e pelas dependências que cruzam esses limites.

#### Separação de fronteiras
- Estabelecer limites entre os componeentes de software, onde um não conheça os detalhes do outro.
- Estabeleça limite entre regras de negócio e detalhes do software.
- Estabelaça limites entre coisas que importam e não importam. Ex: GUI não importa para as regras de negócio, então deve haver um limite entre eles.
- Exemplo: não queremos que as regras de negócio sejam quebradas quando alguem mudar o formato de uma pagina web ou esquema do banco de dados.
- Os componentes de um lado do limite mudam em ritmos diferentes e por razões diferentes quando comparados aos componentes do outro lado do limite.
- Para estabelecer limites em uma arquitetura de software, primeiro particione o sistema em componentes. Alguns desses componentes são regras centrais de negócio; outros são plug-ins que contêm funções necessárias, mas sem relação direta com o negócio central. Em seguida, organize o código desses componentes para que as flechas entre eles aponte em uma única direção - na direção do negócio central.

# Aplicação com baixo acoplamento
- onde há facilidade de trocar as implementações, sem muita dificuldade
- Deixe as regras de negócio, apartadas de frameworks.
- dica: dependa de abstrações (SOLID).

# Niveis
- Componentes de níveis mais altos, aqueles que se encontram longe das entrada e saidas da minha aplicação, devem ser desacoplados de componentes de níveis mais baixos (mais próximos da entrada e saida da minha aplicação).

# Baseada em plugins
- Como os componentes de alto nível não dependente dos componentes de baixo nivel, estes podem ser substituidos tranquilamente.

# Regras de negócio
### Entidades
- São regras e dados cruciais do negócio.
- São classes de alto nível, ou seja, estão mais longes das entradas e saidas.
- Não pode ter nenhuma dependência de framework.
- Contem elementos do negócio, ou seja, representa o negócio.

### Casos de uso
- São regras específicas da aplicação.
- Usam as entidades
- Podem possuir varias classes ou funções
- Especifica a entrada e a saida dos dados, e estes não são as entidades ok.
- Não descreve como o sistema aparece para o usuario, e sim as regras da aplicação que regem sobre a interação entre usuários e entidades, o modo como os dados entram e saem do sistema é irrelevante.
- As entidades não conhecem os casos de uso
- Não pode ter nenhuma dependência de framework.

### Modelos de requisição e resposta
- São entradas e saidas do caso de uso
- Esses modelos não devem possuir nenhum conhecimento, como os dados são enviados ou demonstrados ao usuário.
- Não pode depender de nenhum framework.
- São classes simples

### Adaptadores de interface
- São portas de entrada e saida do core da minha aplicação.

### Camadas e limites
- Dividindo o sistema em fluxos
- componentes de alto nível ter dependencia de abstrações
- tomar cuidado com limites da arquitetura, ou seja, pontos onde não há como o sistema dar opções de implementação. Exemplo: deixar regras de interpretação dos dados em um ui web, em vez de abstrair essa interpretação, onde uma pagina web ou representação para mobile pode implementa-la.

### Pontos para se preocupar
- componentes do software e suas relações
- infraestrutura do software
- estrutura e design do código
- suporte aos requisitos de negócio
- simplificar evoluções no software

# Arquiterua de microserviços
- a principal característica da arquitetura de microserviços é a decomposição da solução em aplicações com uma ou poucas funcionalidades.
- muitas vezes migradas de um sistema monolito, essa arquitetura proporciona maior velocidade na evolução dos domínios de cada microservice.
- o tamanho do microservice é relativo e depende da situação, quando mais fina a granularidade, maior será a malha de microservices.
- diante desse cenário surge novos desafios, necessidades e novos componentes, como:

#### gateway
- um cliente não precisa conhecer o host específico do microservice para acessá-lo, para isso a porta de entrada da nossa malha é um gateway, diante configuração, redirecionará a requisição ao microservice de destino.
#### service descovery
- um servidor que demonstra como está a saúde de nossos microservices, dando a eles um nome (dns), que pode ser utilizado na comunicação entre os microservices.

#### loadbalance
- uma caracteristica dos microservices é que eles não podem guardar estado transacional, pois são construídos para serem desligados ou replicados, um loadbalance faria o papel de distribuição de carga, quando existirem várias instancias dos nossos microservices.

#### configuração externalizada
- o microservice é agnóstico ao ambiente, dessa forma deve funcionar igualmente em dev, hom e prod, respeitando as configurações de cada ambiente inserido. Existem algumas soluções como: configmap do k8s, parameter store da aws, spring config entre outros que nos ajudam nesse ponto.

#### Containers
- são maquindas isoladas que compartilham mesmo kernel, onde nossos microservices podem ser executados.
- são geradas imagens das nossas aplicações e quando as executam transformam em containers.

### Resiliência
- em uma falha de microservice, caso um dependa do resultado do processamento de outro, pode derrubar nossa arquitetura
- o microservice deve ser resiliente, para isso existem alguns patterns que nos ajudam, como:
  - fallback: resposta alternativa a uma falha
  - circuitbreaker: quando aberto falha rapidamente ou emite uma reposta de fallback, quando fechado segui o procedimento padrão. Ele é aberto diante a falha específica(ou não) e número de ocorrências.
  - exemplo de configuração de um circuit breaker
    ```
    resilience4j.circuitbreaker:
      instances:
        product:
          allowHealthIndicatorToFail: false
          registerHealthIndicator: true
          slidingWindowType: COUNT_BASED
          slidingWindowSize: 5
          failureRateThreshold: 50
          waitDurationInOpenState: 10000
          permittedNumberOfCallsInHalfOpenState: 3
          automaticTransitionFromOpenToHalfOpenEnabled: true
          ignoreExceptions:
            - se.magnus.api.exceptions.InvalidInputException
            - se.magnus.api.exceptions.NotFoundException
    ```
  - retry: diante a falha, o microservice que depende da resposta do microservice com erro, repedi em um intervalo de tempo a requisição, com a esperança de receber o resultado com sucesso.
- bulkhead : limitar o número de requisições simultaneas ao microserice.

#### Observabilidade
- como temos várias aplicações, devemos agregar os logs, ter um trace das transações (operação em uma unidade lógica), verificar a saúde, consumo de memória, cpu e etc (podemos utilizar o istio caso nossas apps estejam no k8s).

# ACID
- atomica: a transação somente é finalizada quando completada ou será revertida
- consistência: a transação muda o estado, caso seja revertida, volta o estado anterior
- isolada: uma transação não afeta a outra
- durabilidade: podemos ver as informações em todos os nós, após a confirmação da transação, mesmo após o reinicio dos nós.

# TEOREMA CAP
- consistëncia: os clientes veêm as mesmas informações em todos os nós
- disponibilidade: veremos as informações mesmo que algum nó esteja fora
- tolerância de partição: o cluster continua funcionando, mesmo se ocorrer uma ou mais falhas entre os nós no sistema.

# Dicas para escrever um programa utilizando linguagem funcional
- Nunca use nullvalores. Esqueça que o Scala ainda tem uma nullpalavra-chave.
- Escreva apenas funções puras.
- Use apenas valores imutáveis ​​( val) para todos os campos.
- Cada linha de código deve ser uma expressão algébrica. Sempre que você usar um if, você também deve usar um else.
- Funções puras nunca devem lançar exceções; em vez disso, eles geram valores como Option, Trye Either.
- Não crie “classes” OOP que encapsulam dados e comportamento. Em vez disso, crie estruturas de dados imutáveis ​​usando caseclasses e, em seguida, escreva funções puras que operem nessas estruturas de dados.
- Se você adotar essas regras simples, descobrirá que:
- Seu cérebro vai parar de buscar atalhos para combater o sistema. (Jogar no varcampo ocasional ou na função impura apenas retardará seu processo de aprendizado .)
- Seu código se tornará como álgebra.
- Com o tempo, você entenderá o processo de pensamento do Scala/FP; você descobrirá que um conceito leva logicamente a outro.
 
 
 ## Dicas app java no kubernetes
 - probe liveness apontar para outra porta e não a porta principal da app
 - redinesse levar em consideração as integrações da app, como banco de dados por exemplo.
 - nunca definir memoria e cpu muito baixos na requisição e deixar o limite 25% maior dentro do deployment.

## Quais são as diferenças entre um balanceador de carga, um proxy reverso e um gateway de API?

- Um 𝗹𝗼𝗮𝗱 𝗯𝗮𝗹𝗮𝗻𝗰𝗲𝗿 é um servidor que distribui o tráfego de rede de entrada em vários servidores. O objetivo é garantir que nenhum servidor seja sobrecarregado com tráfego, o que pode levar a tempos de resposta lentos ou até mesmo tempo de inatividade. Os balanceadores de carga são ideais para sites ou aplicativos de alto tráfego que precisam lidar com um grande volume de solicitações.

- Um 𝗿𝗲𝘃𝗲𝗿𝘀𝗲 𝗽𝗿𝗼𝘅𝘆, por outro lado, é um servidor que fica entre o cliente e o servidor da web. O proxy reverso intercepta solicitações de clientes e as encaminha para o servidor apropriado. O proxy reverso também pode armazenar em cache o conteúdo solicitado com frequência, o que pode ajudar a melhorar o desempenho e reduzir a carga do servidor. Os proxies reversos são ideais para sites ou aplicativos que precisam lidar com um grande número de conexões simultâneas.

# spring-data-advanced

- Colocar findall como fetcher para evitar o problema de n+1
- Colocar clear no delete

# Alguns conceitos
- ACID -> transação que fornece atomicidade, consistência, isolamento e durabilidade.

# Uso do @Lock
- para consultas podemos utilizar no método alguns tipos do alocação da tabela
  - @Lock(LockModeType.PESSIMISTIC_READ) -> aloca a tabela imediatamente, não permite multiplos acessos
    - recomenda-se colocar um tempo para alocar: @QueryHints({@QueryHint(name = "javax.persistence.lock.timeout", value = "3000")})
  - @Lock(LockModeType.OPTIMISTIC_FORCE_INCREMENT) -> versiona a entidade, caso ela esteja diferente no momento da atualização, com o versionamento na base de dados, ocorre uma exception e é realizado rollback.

# Uso do isolation
- para transações devemos seguir, dependendo de cada caso, o ACID -> Atomicidade, Consistência, Isolamento e Durabilidade
- Em vez de utilizar o @Lock, podemos utilizar o @Transactional(isolation = Isolation.**), onde é mais seguro e gerenciável diretamente pelo spring data
- temos alguns tipos de isolamento, como:
  - READ_UNCOMMITTED -> permite acessos simultâneos e leitura de dados não comitados.
  - READ_COMMITTED -> permite apenas dados comitados, no entanto se outra transação confirmar os dados, teremos um resultado diferente.
  - REPEATABLE_READ -> permite apenas dados comitados e não demonstra dados comitados em outra transação por um tempo
  - SERIALIZÁVEL -> evita todos os problemas acima, mas limta o acesso simultâneo ao recurso.

# Teorema CAP
- C (consistencia) -> todos leêm as mesmas informações ao mesmo tempo, os dados devem ser replicados a todos os nós
- a (disponibilidade) -> sempre temos respostas da base, mesmo diante a nós inativos.
- P (particionamento) -> cliente deve continuar recebendo os dados, mesmo em ocorra falha em algum nó


- Um 𝗔𝗣𝗜 𝗴𝗮𝘁𝗲𝘄𝗮𝘆 é um servidor que atua como intermediário entre clientes e servidores back-end. O gateway de API é responsável por gerenciar solicitações de API, aplicar políticas de segurança e lidar com autenticação e autorização. Os gateways de API são ideais para arquiteturas de microsserviços, onde vários serviços precisam ser acessados por meio de uma única API.

# Test transações spring
- para simular uma situação real, em produção em um test integrado com o spring, alem de anotar o método com @transaction, devemos anotar com @commit.

# Consulta recursiva com hibernate
```
A Postentidade é mapeada da seguinte forma:

@Entity(name = "Post")
@Table(name = "post")
public class Post {
⠀
    @Id
    private Long id;
⠀
    @Column(length = 100)
    private String title;
}
E a PostCommententidade filha é mapeada assim:

@Entity(name = "PostComment")
@Table(name = "post_comment")
public class PostComment {
⠀
    @Id
    @GeneratedValue(
        generator = "post_comment_seq",
        strategy = GenerationType.SEQUENCE
    )
    @SequenceGenerator(
        name = "post_comment_seq",
        allocationSize = 1
    )
    private Long id;
⠀
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id")
    private Post post;
⠀
    @Column(name = "created_on")
    private LocalDateTime createdOn;
⠀
    @Column(length = 250)
    private String review;
⠀
    private int score;
⠀
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private PostComment parent;
⠀
    @OneToMany(
        mappedBy = "parent",
        cascade = CascadeType.ALL,
        orphanRemoval = true
    )
    private List<PostComment> children = new ArrayList<>();
}
Após mapear as entidades, vamos adicionar as seguintes Postentidades PostComment:

Post post = new Post()
    .setId(1L)
    .setTitle("Post 1");
 
entityManager.persist(post);
 
entityManager.persist(
    new PostComment()
        .setPost(post)
        .setCreatedOn(
            LocalDateTime.of(2024, 6, 13, 12, 23, 5)
        )
        .setScore(1)
        .setReview("Comment 1")
        .addChild(
            new PostComment()
                .setPost(post)
                .setCreatedOn(
                    LocalDateTime.of(2024, 6, 14, 13, 23, 10)
                )
                .setScore(2)
                .setReview("Comment 1.1")
        )
        .addChild(
            new PostComment()
                .setPost(post)
                .setCreatedOn(
                    LocalDateTime.of(2024, 6, 14, 15, 45, 15)
                )
                .setScore(2)
                .setReview("Comment 1.2")
                .addChild(
                    new PostComment()
                        .setPost(post)
                        .setCreatedOn(
                            LocalDateTime.of(2024, 6, 15, 10, 15, 20)
                        )
                        .setScore(1)
                        .setReview("Comment 1.2.1")
                )
        )
);
Hibernate COM consulta RECURSIVA
Nosso caso de uso exige que busquemos uma hierarquia inteira de post_commententidades que descendem de um post_commentregistro de tabela pai fornecido.

Como expliquei neste artigo , podemos usar uma WITH RECURSIVEconsulta SQL nativa para buscar uma hierarquia de registros de tabela, e essa solução funciona bem com qualquer versão do Hibernate.

Entretanto, desde o Hibernate 6, também podemos usar a consulta JPQL para buscar estruturas de dados hierárquicas usando a WITHcláusula, conforme demonstrado pela consulta a seguir:

List<PostCommentRecord>  postComments = entityManager.createQuery("""
    WITH postCommentChildHierarchy AS (
      SELECT pc.children pc
      FROM PostComment pc
      WHERE pc.id = :commentId
⠀
      UNION ALL
⠀
      SELECT pc.children pc
      FROM PostComment pc
      JOIN postCommentChildHierarchy pch ON pc = pch.pc
      ORDER BY pc.id
    )
    SELECT new PostCommentRecord(
        pch.pc.id,
        pch.pc.createdOn,
        pch.pc.review,
        pch.pc.score,
        pch.pc.parent.id
    )
    FROM postCommentChildHierarchy pch
    """, PostCommentRecord.class)
.setParameter("commentId", 1L)
.getResultList();
⠀
assertEquals(3, postComments.size());
assertEquals("Comment 1.1", postComments.get(0).review);
assertEquals("Comment 1.2", postComments.get(1).review);
assertEquals("Comment 1.2.1", postComments.get(2).review);
A consulta WITH RECURSIVE do Hibernate é contraída assim:

. A WITHcláusula usa um alias que podemos referenciar mais abaixo em nossa consulta SQL.
. A primeira consulta dentro da WITHcláusula é chamada de membro âncora e define os registros raiz que serão adicionados à nossa postCommentChildHierarchytabela virtual.
. A segunda consulta dentro da WITHcláusula é chamada de membro recursivo e será repetida até produzir um conjunto de resultados vazio. Os registros produzidos pela execução do membro recursivo serão adicionados à postCommentChildHierarchytabela virtual.
. A última consulta seleciona os registros da postCommentChildHierarchytabela virtual e mapeia os registros para o PostCommentRecord.
```
