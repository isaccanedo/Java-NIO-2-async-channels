## Core Java NIO

# Um guia para canal de soquete assíncrono NIO2

# 1. Visão Geral
Neste artigo, demonstraremos como construir um servidor simples e seu cliente usando as APIs de canal Java 7 NIO.2.

Veremos as classes AsynchronousServerSocketChannel e AsynchronousSocketChannel, que são as classes principais usadas na implementação do servidor e do cliente, respectivamente.

Se você é novo nas APIs do canal NIO.2, temos um artigo introdutório neste site. Você pode lê-lo seguindo este link.

Todas as classes necessárias para usar APIs de canal NIO.2 são agrupadas no pacote java.nio.channels:

```
import java.nio.channels.*;
```

# 2. Como funcionam as APIs de canal assíncrono
As APIs de canal assíncrono foram introduzidas no pacote java.nio.channels existente, simplesmente colocando - prefixando os nomes das classes com a palavra Asynchronous.

Algumas das classes principais incluem: AsynchronousSocketChannel, AsynchronousServerSocketChannel e AsynchronousFileChannel.

Como você deve ter notado, essas classes são semelhantes em estilo às APIs de canal NIO padrão.

E, a maioria das operações de API disponíveis para as classes de canal NIO também estão disponíveis nas novas versões assíncronas. A principal diferença é que os novos canais permitem que algumas operações sejam executadas de forma assíncrona.

Quando uma operação é iniciada, as APIs de canal assíncrono nos fornecem duas alternativas para monitorar e controlar as operações pendentes. A operação pode retornar o objeto java.util.concurrent.Future ou podemos passar para ele um java.nio.channels.CompletionHandler.

# 3. A Abordagem Futura
Um objeto Future representa o resultado de uma computação assíncrona. Assumindo que queremos criar um servidor para ouvir as conexões do cliente, chamamos a API estática aberta no AsynchronousServerSocketChannel e, opcionalmente, vinculamos o canal de soquete retornado a um endereço:

```
AsynchronousServerSocketChannel server 
  = AsynchronousServerSocketChannel.open().bind(null);
```

Passamos nulo para que o sistema possa atribuir um endereço automaticamente. Em seguida, chamamos o método de aceitação no servidor SocketChannel retornado:

```
Future<AsynchronousSocketChannel> future = server.accept();
```

Quando chamamos o método de aceitação de um ServerSocketChannel no IO antigo, ele bloqueia até que uma conexão de entrada seja recebida de um cliente. Mas o método de aceitação de um AsynchronousServerSocketChannel retorna um objeto Future imediatamente.

O tipo genérico do objeto Future é o tipo de retorno da operação. Em nosso caso acima, é AsynchronousSocketChannel, mas também poderia ser Integer ou String, dependendo do tipo de retorno final da operação.

Podemos usar o objeto Future para consultar o estado da operação:

```
future.isDone();
```


Esta API retorna verdadeiro se a operação subjacente já foi concluída. Observe que a conclusão, neste caso, pode significar rescisão normal, uma exceção ou cancelamento.

Também podemos verificar explicitamente se a operação foi cancelada:

```
future.isCancelled();
```

Só retorna verdadeiro se a operação foi cancelada antes de ser concluída normalmente, caso contrário, retorna falso. O cancelamento é realizado pelo método de cancelamento:

```
future.cancel(true);
```

A chamada cancela a operação representada pelo objeto Future. O parâmetro indica que mesmo que a operação tenha sido iniciada, ela pode ser interrompida. Depois que uma operação for concluída, ela não pode ser cancelada

Para recuperar o resultado de um cálculo, usamos o método get:

```
AsynchronousSocketChannel client= future.get();
```

Se chamarmos essa API antes da conclusão da operação, ela será bloqueada até a conclusão e, em seguida, retornará o resultado da operação.

# 4. A abordagem CompletionHandler
A alternativa de usar Future para lidar com operações é um mecanismo de retorno de chamada usando a classe CompletionHandler. Os canais assíncronos permitem que um manipulador de conclusão seja especificado para consumir o resultado de uma operação:

```
AsynchronousServerSocketChannel listener
  = AsynchronousServerSocketChannel.open().bind(null);

listener.accept(
  attachment, new CompletionHandler<AsynchronousSocketChannel, Object>() {
    public void completed(
      AsynchronousSocketChannel client, Object attachment) {
          // do whatever with client
      }
    public void failed(Throwable exc, Object attachment) {
          // handle failure
      }
  });
```

A API de retorno de chamada concluída é chamada quando a operação de E / S é concluída com êxito. O retorno de chamada com falha é invocado se a operação falhar.

Esses métodos de retorno de chamada aceitam outros parâmetros - para nos permitir passar quaisquer dados que consideremos adequados para marcar junto com a operação. Este primeiro parâmetro está disponível como o segundo parâmetro do método de retorno de chamada.

Finalmente, um cenário claro é - usar o mesmo CompletionHandler para diferentes operações assíncronas. Nesse caso, nos beneficiaríamos com a marcação de cada operação para fornecer contexto ao lidar com os resultados, veremos isso em ação na seção a seguir.

# 5. Conclusão
Neste artigo, exploramos os aspectos introdutórios das APIs de canal assíncrono do Java NIO2.