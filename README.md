# Dicas para um código limpo

- Cria-se métodos pequenos
- O nome dos métodos, deve refletir o que ele faz.
- O método deve validar, criar ou consultar, nunca mais de 1 processo junto.
- O método deve possuir  no máximo 3 parâmetros.
- O método deve fazer apenas uma coisa, uma abstração, um propósito.
- Não use parâmetros de saída.
- No código catch, apenas trate a exceção.
- Nunca duplique código.
- Objetos usam abstrações para esconder seus dados ou funções, estratura de dados expões seus dados e não possuem funções.
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
 - OCP : objetivo do sistema ser facil de estender, sem que a mudança gere alto impacto. Para concretizar esse objetivo, particionamos o sistema em compoenentes e organizamos esses componentes em uma hierarquia de dependência que proteja os camponentes de nivel mais alto das mudanças em componentes de nivel mais baixo: Por exemplo: quero proteger o compoente A do componentes B, componente B deve depender do componente A para isso.
 - LSP (substituição de liskov) : capacidade de substituir as entidades filhas pelas pais, para usso usamos interfaces e heranças (polimorfismo).
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
