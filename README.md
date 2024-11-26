# Explicação da prática "TransacoesBancarias" da aula 22
_por Diego Rockenbach_
## Instalação local do Java com VSCode
**Vídeo tutorial nas referências abaixo.**
## Análise do código "TransacoesBancarias"
O código consiste de 3 classes Java:

- ### class Conta
Contém os métodos para manipular os valores de determinada conta bancária (retirar/depositar saldo e checar o saldo total da conta) e quando criado o objeto da classe Conta é passado o saldo inicial que aquela conta contém.
```
class Conta {

  private float saldo = 0f;

  public Conta(float inicial) {
    saldo = inicial;
  }

  public float getSaldo() {
    return saldo;
  }

  public void deposita(float valor) {
    saldo += valor;
  }

  public void retira(float valor) {
    saldo -= valor;
  }

}
```

- ### class Transacoes
Como podem ver abaixo, a classe Transacoes "implements Runnable", ou seja, implementa a interface Runnable. Classes que implementam essa interface caracteristicamente contém o método *run* e, para execução desse, objetos dessa classe deverão ser passados para um objeto Thread. O método run dessa classe é responsável por depositar *valor_credito* e retirar *valor_debito* *n* vezes do objeto *c*, classe Conta. (para uma revisão do conceito de Threads em Java, referencio os slides da aula 21, ou o [W3Schools](https://www.w3schools.com/java/java_threads.asp)).
```
class Transacoes implements Runnable {

  private Conta c;
  private int n;
  private int valor_credito;
  private int valor_debito;

  public Transacoes(Conta c, int n, int valor_credito, int valor_debito) {
    this.c = c;
    this.n = n;
    this.valor_credito = valor_credito;
    this.valor_debito = valor_debito;
  }

  @Override
  public void run() {
    for (int i = 0; i < n; i++) {
      c.deposita(valor_credito);
      c.retira(valor_debito);
    }
  }
}
```

- ### class TestTransacoesBancarias
Como o nome já implica, contém os métodos responsáveis pelos testes e execução principal do código.
```
class TestTransacoesBancarias {

  public static void main(String[] args) {
  
    Conta c = new Conta(0);
    // 1000 depósitos de 10 e 1000 retiradas de 5 = 5000
    Runnable transacoes = new Transacoes(c, 1000, 10, 5);
    
    // Cria 2 objetos da classe Thread
    Thread thread1 = new Thread(transacoes); // 5000
    Thread thread2 = new Thread(transacoes); // 5000

    // Ativa execução das threads
    thread1.start();
    thread2.start();

    // Aguarda término das threads
    try {
      thread1.join();
      thread2.join();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }

    // Saldo deveria ser 10000
    System.out.println("Saldo final: " + c.getSaldo());
  }
}
```

## Problema no código
O saldo inicial da conta criada no código é de R$ 0, e então o runnable Transacoes é configurado para realizar 1000 depósitos de R$ 10 seguido por 1000 retiradas de R$ 5 (R$ 0 + R$ 10000 - R$ 5000 = R$ 5000). Então são criados dois objetos da classe Thread, e logo em seguida eles são iniciados com o comando *.start()*, e então alinhados em seu final com o comando *.join()*. Visto que ambas foram executadas alterando o valor do objeto *c* da classe Conta, o saldo total da conta após todas as operações (obtido com o método *getSaldo()*) deveria ser de R$ 10000, correto?

Na teoria sim, mas na prática o valor é inconsistente devido ao acesso simultâneo das threads aos métodos *deposita* e *retira*, da classe Conta. Visto que ambas as threads estão acessando o mesmo saldo do mesmo objeto simultaneamente e realizando diversas operações com ele, o valor é vítima da falta de sincronização dessas duas threads e, portanto, raramente resulta nos esperados R$ 10000.

## Solução
É simples.

Basta adicionar aos métodos responsáveis pela alteração dos valores do objeto da classe Conta a keyword **synchronized**. Assim, a classe Conta ficará deste modo:

```
class Conta {

  private float saldo = 0f;

  public Conta(float inicial) {
    saldo = inicial;
  }

  public float getSaldo() {
    return saldo;
  }

  public synchronized void deposita(float valor) {
    saldo += valor;
  }

  public synchronized void retira(float valor) {
    saldo -= valor;
  }

}
```
### Mas por quê isso funciona?

Cada objeto em Java possui um monitor (ou lock) interno. O synchronized usa esse monitor para implementar a exclusão mútua, ou seja, quando uma thread entra em um método sincronizado, ela adquire o lock do monitor do objeto. Enquanto a thread tiver o lock, nenhuma outra thread pode acessar qualquer código sincronizado que use o mesmo lock. O lock é automaticamente liberado quando a thread sai do código sincronizado, seja por término normal ou por exceção.

Não é necessário adicionar o synchronized ao método getSaldo() ou ao criador do objeto Conta() pois eles não são acessados por threads e, portanto, não correm risco de acessos simultâneos.



### Material usado

> [Instalação de Java no Windows VSCode](https://www.youtube.com/watch?v=fbyobdxDQno)

> Slides da matéria
