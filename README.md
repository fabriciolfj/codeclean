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
- a principal característica da arquitetura de microserviços é a decomposição ou aplicações com um ou poucas funcionalidades.
- muitas vezes migradas de um sistema monolito, essa arquitetura proporciona maior velocidade na evolução dos domínios de cada microservice.
- o tamanho do microservice é relativo e depende da situação, quando mais fina a granularidade, maior será minha malha de microservices.
- diante desse cenário surge novos desafios, necessidades e novos componentes, como:

#### gateway
- um cliente não precisa conhecer o host específico do microservice para acessá-lo, para isso a porta de entrada da nossa malha é um gateway, diante configuração, redirecionará a requisição ao microservice de destino.
#### service descovery
- um servidor que demonstra como está a saúde de nossos microservices, dando a eles um nome (dns), que pode ser utilizado na comunicação entre os microservices.

#### loadbalance
- uma caracteristica dos microservices é que eles não podem guardar estado transacional, pois são construídos para serem desligados ou replicados, um loadbalance faria o papel de distribuição de carga, quando existirem várias instancias dos nossos microservices.

#### configuração externalizada
- o microservice é agnóstico ao ambiente, dessa forma deve funcionar igualmente em dev, hom e prod, respeitando as configurações de cara ambiente inserido. Existem algumas soluções como: configmap do k8s, parameter store da aws, spring config entre outros que nos ajudam nesse ponto.

#### Containers
- são maquindas isoladas que compartilham mesmo kernel, onde nossos microservices podem ser executados.
- são geradas imagens das nossas aplicações e quando as executam transformam em containers.

### Resiliência
- em uma falha de microservice, caso um dependa do resultado do processamento de outro, uma falha não pode derrubar nossa arquitetura
- o microservice deve ser resiliente, para isso existem alguns patterns que nos ajudam, como:
  - fallback: resposta alternativa a uma falha
  - circuitbreaker: quando aberto falha rapidamente ou emite uma reposta de fallback, quando fechado segui o procedimento padrão. Ele é aberto diante a falha específica e número de ocorrências.
  - retry: diante a falha, o microservice que depende da resposta do microservice com erro, repedi em um intervalo de tempo a requisição, com a esperança de receber o resultado com sucesso.
- bulkhead : limitar o número de requisições simultaneas ao microserice.

#### Observabilidade
- como temos várias aplicações, devemos agregar os logs, ter um trace das transações (operação em uma unidade lógica), verificar a saúde, consumo de memória, cpu e etc (podemos utilizar o istio caso nossas apps estejam no k8s).
