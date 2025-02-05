# Thread vs Processo

**Processo** é um programa independente com seu espaço na memória.
**Thread** é um sub-processo leve que compartilha espaço de memória com seu processo "pai".

## Thread têm um ciclo de vida, que segue a ordem:

- New: Criada, mas não foi iniciada.
- Runnable: Está pronta para iniciar, após start()
- Blocked/Waiting: Esperando um 'monitor lock'
    - Monitor: É uma construção para a sincronização que permite as threads sejam tanto de exclusão mútua quanto de cooperação
      Além dos dados implementaram uma trava, cada objeto java é logicamente associado a dados que utilizam um conjunto de espera (wait-set).
    - Lock: É um tipo de dado que faz parte da memória heap, cada objeto na JVM possui esse 'lock', e quaquer programa pode utilizar a mesma para
      coordenar o acesso multi-thread ao objeto. Se utiliza das palavras reservadas synchronized, wait(), notify().
- Timed Waiting: Procesos 'dorme' por determinado tempo, utilizando Thread.sleep() ou wait(timeout).
- Terminated: Finalizou o tempo de execução

## Para criar uma thread, dois métodos são mais utilizados:

### Extendendo-se a Thread:

```java
    class MyThread extends Thread {
        public void run() {
            System.out.println("Thread rodando");
        }
    }
    MyThread t = new MyThread();
    t.start();
```

### Ou implementando a interface Runnable, que é mais flexível:

```java
    Runnable task = () -> System.out.println("Thread Runnable");
    Thread t = new Thread(task);
    t.start();
```
## Sincronicação

### Principal problema: Resultados imprevisíveis
São causados por race conditions, quando duas threads ou mais tentam acessar e manipular um recurso de forma concorrente. 
isso é solucionado com a sincronização das threads, o método mais comum é com syncronized. Isso também é enfatizado na documentação da Oracle na seção de Synchronized Methods: https://docs.oracle.com/javase/tutorial/essential/concurrency/syncmeth.html

> To make a method synchronized, simply add the synchronized keyword to its declaration:

```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```
Agora, duas invocações ao método sincronizado serão impossíveis de serem realizadas no mesmo objeto. Enquanto uma thread executar, todas as outras que invocam esse método estarão em Timed Waiting enquanto aguardam a liberação do lock.
Além disso, quando o método sincronizado finalizar, automaticamente seta uma relação de happens-before com qualquer método subsequente chamado para o mesmo objeto, isso garante que manipulações dos serãos são visives para todas as outras threads.

### Outras abordagens incluem:

 - ReentrantLock: da bibliocata java.util.concurrent.locks, permite locks explíceitos e desbloqueios manuais no objeto.
 - AtomicInteger: da biblioteca java.util.concurrent.atomic, utilziado em variaveís númericas que precisam de sincronização, mais eficiente que simples counts.

## Problema de consumer-producer

Problema clássico de sincronização, um **producer** irá gerar dados e enviar à uma fila ou buffer compartilhado, e um **consumer** irá remover os dados dessa fila ou buffer e processa-los.

Problema gira em torno de com o producer e o consumer não devem ter acesso ao buffer compartilhado ao mesmo tempo, e também a questão dos bounded buffers, onde o producer deve esperar se o vuffer estiver cheio, assim como o consumer deve eseprar se o buffer estiver vazio.

### Solução mais direta e simples: BlockingQueue

BlockingQueue irá lidar automaticamente com a questão de sincronização, irá bloquear as threads quando o proceuder tentar adicionar algo à um buffer cheio, e também bloquear a thread que o consumer tentar retirar algo de um buffer vazio.

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class ProducerConsumerExample {
    public static void main(String[] args) {
        BlockingQueue<Integer> buffer = new LinkedBlockingQueue<>(5); // Capacity 5

        // Producer Thread
        Runnable producer = () -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    buffer.put(i); // Blocks if queue is full
                    System.out.println("Produced: " + i);
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        // Consumer Thread
        Runnable consumer = () -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    int value = buffer.take(); // Blocks if queue is empty
                    System.out.println("Consumed: " + value);
                    Thread.sleep(200);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        new Thread(producer).start();
        new Thread(consumer).start();
    }
}
```


 ## Perguntas gerais

 - Como evitar race conditions?
   → Use synchronized, locks ou classes atômicas.

 - Qual a diferença entre wait() e sleep()?
   → wait() libera o lock; sleep() não libera.

 - O que é deadlock? Como depurar?
   → Use jstack para analisar dumps de thread.

 - Quando usar volatile?
   → Quando uma variável é compartilhada e atualizada apenas atomicamente (ex: flags).

 - Explique o problema do produtor-consumidor.
   → Use BlockingQueue ou wait()/notify().
