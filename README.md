# troubleshooting-java
- para investigar problemas em app, devemos utilizar ferramentas criadoras de perfis
  - por exemplo temos o visualvm, para monitoramente e análise de desempenho para app java.
  - caso tenha problemas como codigo 62, utiliza como arg na vm do app -Xverify:none
- devemos criar perfis com base em uma amostragem e não da base de codigo por um todo.

## problema nas consultas n+1
- Quando uma estrutura precisa obter dados de várias tabelas, ela geralmente sabe compor uma consulta e obter todos os dados em uma única chamada. No entanto, se você não usar o framework corretamente, ele pode pegar apenas parte dos dados com uma consulta inicial e então, para cada registro inicialmente recuperado, executar uma consulta separada. Assim, ao invés de executar apenas uma consulta, o framework enviará uma consulta inicial mais N outras (uma para cada um dos N registros recuperados pela primeira); chamamos isso de problema de consulta N+1, que geralmente cria uma latência significativa executando muitas consultas em vez de apenas uma.  

## Vazamento de memória
- posíveis causas:
  - quando estamos referenciando objetos que não são mais usados
  - threads abertas que não terminam seu processo
  - GC não gasta nenhum recurso de CPU. Isso também não é um bom sinal. Em outras palavras, o aplicativo gasta muito poder de processamento, mas não processa nada. Esses sinais geralmente indicam threads zumbis, que geralmente são consequência de problemas de simultaneidade.
  - quando ocorre um outofmemory error, precisamos de um head dump para verificar a causa
 
## desempenho
- verifique o tempo de execução e o tempo de cpu, caso aja tempo de execução e o tempo de cpu seja 0, o app está esperando por algum

## Soluções paliativas para o vazamento de memória  
- aumentar a memoria head: -Xmx1G por exemplo
- aumentar o metaspace (local aonde ficam os metadados das classes, metodos, campos da classe, nome da classe, pacote, modificadores de acesso e etc, necessários para a execução do app): -XX:MaxMetaspaceSize=100M
  - obs: situaões aonde o metaspace é totalmente preenchido, é uma grande volume no uso de reflexão.
 
## JProfiler
- possui alguns recursos a mais que o visualVM, como:
  - grafico de chamadas
  - identificador se sua app está fechando corretamente as conexões com uma base relacional
  - gráfica de chamas, demostra a pilha de chamadas 

## metodo esperando por si mesmo
- quando a sua thread foi travada

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

- arquietura de um sistema é definida pelos limites específicos entro desse sistema e pelas dependências que cruzam esses limites.

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

# Dias para escrever um programa utilizando linguagem funcional
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

- Um 𝗔𝗣𝗜 𝗴𝗮𝘁𝗲𝘄𝗮𝘆 é um servidor que atua como intermediário entre clientes e servidores back-end. O gateway de API é responsável por gerenciar solicitações de API, aplicar políticas de segurança e lidar com autenticação e autorização. Os gateways de API são ideais para arquiteturas de microsserviços, onde vários serviços precisam ser acessados por meio de uma única API.
