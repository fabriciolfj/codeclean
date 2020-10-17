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
