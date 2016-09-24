---
author: Nicolas Perez Santoro
categories:
- jvm
- framework
- java
- concurrency
comments: true
date: 2012-02-29T00:00:00Z
title: akka, un framework con actores en la jvm
url: /2012/02/29/akka-un-framework-con-actores-en-la-jvm/
---

## Introducción

[Akka](http://akka.io/) es un framework para programación concurrente, distribuida y tolerante a fallos en la JVM, principalmente basado en Scala pero que también soporta Java. Akka implementa el modelo de Actores ([http://en.wikipedia.org/wiki/Actor_model](http://en.wikipedia.org/wiki/Actor_model)) que se basa en entidades llamadas *Actores* que se ejecutan concurrentemente y se envían mensajes de forma asincrónica entre sí. Este mismo modelo es el que implementa el lenguaje de programación Erlang. Akka también implementa Software Transactional Memory, un mecanismo de memoria transaccional que es una alternativa al locking para acceder a estructuras de datos en memoria compartidas, y que es similar al que está implementado en el lenguaje Clojure.

En este articulo, veremos como utilizar Akka en Java, utilizando Actores y STM para resolver un problema de concurrencia simple pero que involucra multiples variables que se actualizan a la vez, con un cierto nivel de contención. Vamos a ver varias implementaciones, explorando las posibilidades de Akka, con el código fuente disponible para bajar. Utilizaremos la versión Akka 1.2.

<!--more-->

## Actores

Los actores son entidades que ejecutan concurrentemente y se envían mensajes asincronicos entre sí (si bien es posible implementar mensajes sincronicos). El modelo de actores está pensado en base a la idea de que los actores no comparten estado (aunque más adelante veremos una forma de que compartan estado), y que de esta manera no hay problemas de locking para acceder al estado: El estado es único a cada actor, y cada actor solo procesa un mensaje a la vez. El modelo de actores es inherentemente distribuido: Como cada actor no comparte estado con el resto de los actores, estos pueden estar ubicados en diferentes JVMs en diferentes nodos, y que estén en la misma JVM puede verse como un caso particular, o una optimización.

Los actores en Akka son extremadamente livianos. Por cada actor, el framework Akka asigna unos 600 bytes y por defecto cada actor no corre necesariamente en un thread de la JVM, sino que Akka implementa una planificación propia al estilo *green thread*, esto permite tener una gran cantidad de Actores corriendo en una JVM. De todas maneras esto es configurable y es posible asignar un thread *dedicado* a un actor.

## Software Transactional Memory

El mecanismo de Software Transactional Memory, o STM, es un mecanismo que permite acceder a variables transacccionales dentro de una transacción con las propiedades ACI de ACID (sin la D de durabilidad, porque es memoria volátil): Atomicidad, Consistencia, Aislamiento (Isolation). Lo unico que hay que hacer es envolver en una transacción los accesos a las variables (ya sea lectura o escritura).

La memoria transaccional tiene varias ventajas con respecto al locking:

- No puede haber deadlocks.
- Las transacciones pueden componerse: Es posible tomar dos operaciones transaccionales y combinarlas y envolverlas en una transacción. Esto no se puede hacer de una manera simple utilizando unicamente locks.
- Permite aprovechar al maximo el paralelismo porque al no haber locks, se reduce la contención (si bien es posible generar una congestión en el acceso a una única variable).

## Un problema a resolver: Moving Robots

El problema que vamos a resolver está inspirado en una demostración del mecanismo de STM de Clojure, llamado "ants-demo", que consiste en una simulación de hormigas que se mueven por un mundo, moviendose y comiendo comida, donde todas las hormigas funcionan concurrentemente alterando el mundo, y donde cada tanto debe mostrarse por pantalla el estado actual del mundo de forma consistente. Nuestro problema es más sencillo, e involucra una cuadrilla con casilleros de NxM, donde cada casillero puede estar vacío o tener un robot, y concurrentemente cada cierto tiempo los robots tratan de moverse en una de las 4 direcciones, si hubiera en el casillero destino otro robot ya ubicado, el robot se "detiene" por la presencia de su colega robot, y no avanzará. 

Claramente es necesario tener una vista consistente y ordenada para acceder al tablero en todo momento, porque pueden suceder cosas como que dos robots traten de acceder al mismo tiempo a un mismo casillero, y solo uno tiene que moverse. A a su vez es importante poder leer el tablero de forma consistente para mostrarlo en pantalla, esto es necesario porque no queremos que mientras que vamos leyendo el tablero para mostrarlo se vaya actualizando y aparezcan robots duplicados (porque cuando leimos el primer casillero, el robot estaba ahi, y cuando leimos el segundo, el robot ya se movio) o robots invisibles (porque cuando leimos el primer casillero, el robot no estaba ahi, pero luego se movio a esa posicion cuando leimos el segundo donde estaba originalmente).

Existen entonces básicamente dos tipos distintos de cambios de estado y consulta del sistema:

1. Movimiento de un robot a otro casillero, el cual incluye una consulta al estado del casillero destino (si tiene o no tiene un robot), y en base a esa consulta se decide si moverse o no. Esto no es tan simple como parece, porque tenemos que asegurarnos que si consultamos el estado y este no tiene un robot, al momento de asignar nuestro robot ese estado no puede ser modificado en el medio.

2. Consultar el estado del tablero en su integridad, consistentemente. Esto se hace cada cierto tiempo y con una menor frecuencia que el movimiento de los robots.

La forma más simple de lidiar con este problema de concurrencia es simplemente tener un lock global para todo el tablero, cada vez que se va a mover un robot, o cada vez que se va a mostrar por pantalla. El problema de este esquema es que a medida que el tablero crece y la cantidad de robots crece, hay cada vez más contención sobre ese lock, cuando en realidad, para mover un robot a un casillero, solamente nos interesan dos casilleros: el casillero origen y el casillero destino (para mostrarlo, en ese caso sí interesa la totalidad de los casilleros).

Otra alternativa es utilizar locking casillero a casillero, en este caso el problema es evitar los deadlocks, cosa que no es tan trivial ya que tanto para mover un robot como para mostrarlo se necesitarían varios locks. Nosotros vamos a intentar resolverlo en Akka, sin locking.

## Primer implementación, actores no tipados sin STM

En Akka hay dos tipos de actores, actores no tipados y actores tipados. Vamos a utilizar primero los actores no tipados, que son aquellos que heredan de la clase `UntypedActor` y solo deben implementar el método `onReceive`, que recibe un objeto mensaje por parámetro. El mensaje es un POJO, que no tiene ninguna restricción, excepto que es inmutable (aunque como veremos más tarde, hay una forma de relajar esta restricción).

Definimos entonces dos tipos de actores, `Robot` y `Casillero`. Cada `Robot` conoce a su casillero actual, y cada `Casillero` conoce a su `Robot` actual, y a sus vecinos en cada una de las 4 direcciones. Cada `Robot` tiene un nombre que es un string que se utiliza para poder mostrar el tablero en pantalla, imprimiendo el nombre del robot en cada casillero o un "#" si el casillero esta vacío.

Cada actor puede responder a los siguientes mensajes:

### Robot

* `MoveteEnDireccion`: Al recibir este mensaje, el robot se moverá en una dirección especificada dentro del mensaje. En el loop principal del programa habrá un thread que manda estos mensajes a todos los actores Robot en determinados intervalos de tiempo, con direcciones aleatorias.

* `DameTuNombre`: Devuelve el nombre del robot, para poder imprimir su nombre.

* `NuevoCasillero`: Contiene una referencia al Casillero el cual será su nuevo casillero, utilizado tanto para inicializar la posición del robot como respuesta al movimiento del robot, en ambos casos actualiza el casillero actual del robot.

### Casillero

* `MovemeA`: Mensaje que instruye que el robot contenido en el casillero, debe moverse hacia una direccion especificada en el mensaje.

* `RecibiRobot`: Mensaje que contiene una referencia a un robot, y debe asignarse como el robot actual del casillero, respondiendo con un ResultadoMovimiento si pudo asignarlo o no (porque ya tenia otro robot actualmente).

* `ResultadoMovimiento`: Mensaje que unicamente se recibe como respuesta a un RecibiRobot.

* `DameTuRobot`: Devuelve el robot actualmente contenido, necesario para mostrar el tablero en pantalla.

* `TusVecinosSon`: Inicializa los vecinos del casillero en cada direccion.

Entonces, el mecanismo por el cual los robots se mueven es el siguiente:

1. El robot recibe el mensaje MoveteEnDireccion.
2. El robot un mensaje MovemeA al Casillero, con la dirección. Lo envía de forma sincronica, es decir, el actor robot se bloquea esperando una respuesta y no puede recibir más mensajes.
3. El casillero envía el mensaje RecibiRobot al casillero vecino que se encuentra en esa posicion (si hubiera alguno, ya que en caso de estar contra una pared), y también de forma sincronica espera una respuesta.
4. El casillero vecino que recibe el mensaje RecibiRobot, si no tiene ningun robot, lo asigna como el robot actual, y responde ResultadoMovimiento con true dentro, o con false en caso contrario.
5. El casillero original recibe la respuesta del casillero vecino, y responde al robot con un mensaje NuevoCasillero con el casillero vecino, si pudo moverse a dicho casillero, o con el mismo casillero original, si no pudo moverse.
6. El robot recibe el mensaje de respuesta, y actualiza su casillero actual.




``` java Robot.java

public class Robot extends UntypedActor {
  private final String nombre;
  private ActorRef casillero;

  public Robot(String nombre) {
    this.nombre = nombre;

  }

  public void onReceive(Object message) throws Exception {
    if (message instanceof MoveteEnDireccion) {
      MoveteEnDireccion moveteEnDireccion = (MoveteEnDireccion) message;
      nuevoCasillero((NuevoCasillero) casillero.ask(new MovemeA(moveteEnDireccion.getDireccion())).get());
    } else if (message instanceof DameTuNombre) {
      getContext().channel().tell(this.nombre);
    }else if (message instanceof NuevoCasillero) {
      nuevoCasillero((NuevoCasillero) message);
    } else {
      throw new IllegalArgumentException("Unknown message: " + message);
    }
  }

  private  void nuevoCasillero(NuevoCasillero nuevoCasillero) {
    this.casillero = nuevoCasillero.getNuevoCasillero();
    }

}

```

``` java Casillero.java

public class Casillero extends UntypedActor {
  private ActorRef robotEnCasillero;
  private Map<Direccion, ActorRef> casillerosVecinos;

  @Override
  public  void onReceive(Object message) throws Exception {
    if (message instanceof MovemeA) {
      movemeA((MovemeA) message);
    } else if (message instanceof RecibiRobot) {
      recibiRobot((RecibiRobot) message);
    } else if (message instanceof DameTuRobot) {
      dameTuRobot();
    } else if (message instanceof TusVecinosSon) {
      tusVecinosSon((TusVecinosSon) message);
    } else {
      throw new IllegalArgumentException("Unknown message: " + message);
    }
  }

  private  void tusVecinosSon(TusVecinosSon tusVecinosSon) {
    this.casillerosVecinos = tusVecinosSon.getCasillerosVecinos();
  }

  private  void dameTuRobot() {
    getContext().reply(new MiRobot(this.robotEnCasillero));
  }

  private  void recibiRobot(RecibiRobot recibiRobot) {
    if (robotEnCasillero == null) {
      robotEnCasillero = recibiRobot.getRobot();
      getContext().channel().tell(new ResultadoMovimiento(true));
    } else {
      getContext().channel().tell(new ResultadoMovimiento(false));
    }
  }

  private  void movemeA(MovemeA movemeA) {
    Direccion haciaDonde = movemeA.getDireccion();
    ActorRef casilleroDestino = getCasilleroVecinoPorDireccion(haciaDonde);
    ActorRef nuevoCasillero = intentarMoverRobot(robotEnCasillero, casilleroDestino);
    getContext().reply(new NuevoCasillero(nuevoCasillero));
  }

  private ActorRef intentarMoverRobot(ActorRef robot, ActorRef casilleroDestino) {
    if (casilleroDestino == null) {
      return getContext();
    }
    ResultadoMovimiento resultadoMovimiento = (ResultadoMovimiento) casilleroDestino.ask(new RecibiRobot(robot))
            .get();
    if (resultadoMovimiento.fueExitoso()) {
      this.robotEnCasillero = null; // el robot ya no va mas aca
      return casilleroDestino;
    } else {
      return getContext();
    }
  }

  private ActorRef getCasilleroVecinoPorDireccion(Direccion haciaDonde) {
    return this.casillerosVecinos.get(haciaDonde);
  }
}

```

*Nota: Las referencias entre actores no tipados en Akka se hacen mediante un ActorRef, que es un objeto que permite referenciar a un actor y comunicarnos con él. El objeto real es instanciado por Akka y no tenemos acceso directo a él. El mecanismo de ActorRef es serializable y hasta incluso puede enviarse serializadamente a otra JVM y esta puede utilizarla y enviar mensajes al Actor correcto.*

¿Como funciona esto? En primer lugar hay que tener en cuenta que cada actor solamente puede estar procesando un mensaje a la vez, y cuando inicia una comunicación sincronica, bloquea su procesamiento hasta que reciba la respuesta de la comunicación sincronica. Como solo el actor puede actualizar su propio estado, en cierta forma podemos pensar que este mecanismo funciona como un mecanismo de locking, ya que el casillero original no atenderá más mensajes hasta no recibir la respuesta del casillero vecino.

Esta forma de resolverlo, de todas maneras, es insuficiente. En primer lugar, porque no es posible consultar una vista del tablero de forma consistente: La forma es preguntarle a los casilleros cuales son sus robots y obtener una vista consistente, ya que puede haber mensajes dando vueltas que estén actualizando el tablero, y podríamos sufrir el problema de tener robots duplicados o invisibles, como ya explicamos. En segundo lugar, **teoricamente sería posible tener deadlocks**: si bien no lo reproduci, podría suceder que un robot quiera moverse a la derecha cuando a su derecha hay otro robot queriendo moverse a la izquierda. Si ambos casilleros enviaran un mensaje de recibi robot al mismo tiempo al otro lado, estarían bloqueandose mutuamente esperando la respuesta, hasta que ambos terminen con timeout. **Esta es una de las razones por la cual hay que ser muy cuidadosos con la comunicación sincronica entre actores, y en general hay que preferir la asincronica.**

Como dice la documentación de Akka, cuando hay que actualizar estado compartido entre actores de forma consistente, **no es un problema fácil de resolver**. Akka ofrece un mecanismo que se llama Transactors que permitirían modificar el estado de varios actores al mismo tiempo. Dicho mecanismo se intentó implementar para este ejercicio, pero no había una forma de tomar decisiones en base al estado de uno de los actores (el casillero destino) cuando influyen en las decisiones del estado de otros actores (el robot y el casillero destino).
Otra solución posible dentro de Akka, es usar STM, y este es el segundo intento.

## Segunda implementación, actores no tipados con STM

No debería sorprendernos que la forma ideal de lidiar con este problema sea utilizando STM, ya que este ejercicio está basado en un ejemplo de utilización de STM en Clojure. Con STM, lo que hacemos es utilizar una estructura de datos que gira en torno a variables transaccionales, y se consulta y se actualiza con transacciones.

Lo que hacemos entonces es una clase Tablero que contiene una variable de tipo `Ref[][]`: es decir, un array de dos dimensiones de tipo `Ref`. `Ref` es la variable transaccional de Akka, y es una variable que podemos consultar mediante get y set dentro de una transacción, y siempre tendremos una visión consistente de estas variables. Este array contendrá el nombre de cada robot, o null si no hubiera robot. Y existen dos transacciones que se aplicarán sobre este tablero: Una es mover un robot, dado una Posicion origen y una dirección, y la otra es obtener una vista del tablero, que en este caso es simplemente construir un String (hay una tercera para inicializar el tablero, pero solamente se utiliza a modo de set up).

``` java Tablero.java

public class Tablero {
  private Ref<String>[][] casilleros;
  private final  int xLength;
  private final  int yLength;

  public Tablero(int xLength, int yLength) {
    this.xLength = xLength;
    this.yLength = yLength;
    this.casilleros = new Ref[xLength][yLength];
    for(int i = 0; i < xLength; i++){
      for(int j = 0; j < yLength; j++){
        this.casilleros[i][j] = new Ref<String>();
      }  
    }
  }
  public  void putRobotAt(final Posicion posicion, final String robotName){
    new Atomic<Object>(){

      @Override
            public Object atomically() {
        casilleros[posicion.getX()][posicion.getY()].set(robotName);
        return null;
            }}.execute();
  }

  public Posicion moveRobot(final Posicion posicionOrigen, final Direccion direccion) {
    return new Atomic<Posicion>() {

      @Override
      public Posicion atomically() {
        int x = posicionOrigen.getX();
        int y = posicionOrigen.getY();
        Ref<String> casilleroOrigen = casilleros[x][y];
        Posicion posicionDestino = null;
        switch (direccion) {
        case ABAJO:
          if (x + 1 < xLength) {
            posicionDestino = new Posicion(x + 1, y);
          }
          break;
        case ARRIBA:
          if (x - 1 >= 0) {
            posicionDestino = new Posicion(x - 1, y);
          }
          break;
        case DERECHA:
          if (y + 1 < yLength) {
            posicionDestino = new Posicion(x, y + 1);
          }
          break;
        case IZQUIERDA:
          if (y - 1 >= 0) {
            posicionDestino = new Posicion(x, y - 1);
          }
          break;
        }
        if (posicionDestino == null) {
          return posicionOrigen;
        }
        Ref<String> casilleroDestino = casilleros[posicionDestino.getX()][posicionDestino.getY()];
        if (casilleroDestino.get() != null) {
          return posicionOrigen;
        }
        casilleroDestino.set(casilleroOrigen.get());
        casilleroOrigen.set(null);
        return posicionDestino;
      }

    }.execute();
  }

  public String toString() {
    return new Atomic<String>() {

      @Override
      public String atomically() {
        StringBuilder tablero = new StringBuilder();
        for (int i = 0; i < xLength; i++) {
          StringBuilder fila = new StringBuilder();
          for (int j = 0; j < yLength; j++) {
            String robotName = casilleros[i][j].get();
            if (robotName == null) {
              robotName = "#";
            }
            fila.append(robotName);
          }
          tablero.append(fila);
          tablero.append("\n");
        }
        return tablero.toString();
      }
    }.execute();

  }
}

```

Este objeto es totalmente thread-safe, y como explica la documentación de Akka, al utilizar STM podemos compartir estado entre Actores, aunque obviamente, deben residir en la misma JVM.
Descartamos entonces el actor Casillero, y nos quedamos unicamente con el actor Robot, que se implementa simplemente así:

``` java Robot.java

public class Robot extends UntypedActor {
  private Posicion posicion;
  private final Tablero tablero;

  public Robot(Posicion posicion, Tablero tablero) {
    this.posicion = posicion;
    this.tablero = tablero;
  }

  public  void onReceive(final Object message) throws Exception {
    if (message instanceof MoveteEnDireccion) {
      MoveteEnDireccion moveteEnDireccion = (MoveteEnDireccion) message;
      this.posicion = this.tablero.moveRobot(this.posicion, moveteEnDireccion.getDireccion());
    } else {
      throw new IllegalArgumentException("Unknown message: " + message);
    }
  }
}

```

Todos los robots tienen una referencia al mismo tablero, y actualizan este estado utilizando transacciones. Luego, por afuera se consulta sobre el mismo objeto el método para obtener la vista. Esto resuelve efectivamente el problema planteado.

¿Que más hay que tener en cuenta? Las transacciones modifican las mismas variables, entonces, ¿como hace Akka para resolver problemas de concurrencia? Utiliza **multiversionado** de variables, donde existen varias versiones del estado de las variables, consultado de forma consistente, pero también detecta deadlocks y reinicia transacciones, puede reiniciarlas porque las transacciones deben ser totalmente libres de side effects (más allá de alterar las Ref, y las Ref solamente deben apuntar a objetos inmutables), como es el caso de las que implementamos aquí. Ahora bien, **si subimos el numero de robots o el numero de movimientos iniciados, podemos llegar a ver que el programa deja de funcionar ya que las transacciones se reinician demasiadas veces**, hay que tener cuidado con esta posibilidad, y potencialmente configurar las transacciones de manera distinta.

## Tercer implementación, Actores tipados

Esta implementación es esencialmente identica a la de actores no tipados, la diferencia está en la forma de implementar los actores. Con actores no tipados, los actores implementan el método onReceive y reciben los mensajes como objetos, heredan de UntypedActor, y se referencia con ActorRef. Con actores tipados, los actores definen e implementan una interfaz donde cada método de la interfaz es un posible mensaje que puede recibir el objeto, heredan de TypedActor, y se referencian mediante la interfaz: Akka utiliza [aspectos](http://en.wikipedia.org/wiki/Aspect-oriented_programming) para generar un proxy al Actor, entonces invocar a un método del actor lo que hace es generar un mensaje asincrónico por detrás si el método es void, o sincrónico si el método devuelve un valor (hay una tercera opción que es devolver un Future).

Así queda implementado el actor Robot utilizando actores tipados:

``` java Robot.java

public interface Robot {
  void moveteEnDireccion(Direccion direccion);
}

```

``` java RobotImpl.java

public class RobotImpl extends TypedActor implements Robot {
  private Posicion posicion;
  private final Tablero tablero;

  public RobotImpl(Posicion posicion, Tablero tablero) {
    this.posicion = posicion;
    this.tablero = tablero;
  }

  @Override
    public  void moveteEnDireccion(Direccion direccion) {
    this.posicion = this.tablero.moveRobot(this.posicion, direccion);
  }
}

```

¿Cual es la gran diferencia entre ambos esquemas? Actores tipados parece más fácil de utilizar, es cuestion de definir una interfaz y unos métodos, no hace falta crear clases para los mensajes, e invocar métodos de un Actor es muy similar a invocar métodos de un objeto. Pero los actores no tipados parecen ser más flexibles: Podemos consultar muchas más cosas sobre ellos, podemos hacer forwarding de mensajes de un actor a otro (pasar el mensaje a otro actor y que el actor destino responda directamente a quien envió el mensaje original), ya que el mensaje es un objeto, y más.

## Cuarta implementación, Actores remotos

En esta implementación, quien muestra el estado del tablero es un proceso cliente, y quien tiene al tablero y a los actores Robot es un proceso servidor. Hay otro actor, muy simple llamado `VistaTablero` con un método `getVista()` que devuelve un `String[][]` con el estado actual del tablero. Este actor se registra como un servicio con nombre "vista-tablero", y se escucha en un puerto determinado, el cliente se conecta al servidor y obtiene una referencia a ese actor. Una vez obtenida la referencia, utilizar ese Actor es como si estuviera en la misma JVM. Esto es un mecanismo muy poderoso y fácil de utilizar.

El servicio se registra en el servidor así:

``` java

Actors.remote().start("localhost", 2552).registerTypedActor("vista-tablero", vistaTablero);

```

y se obtiene en el cliente de esta forma:

``` java

VistaTablero vistaTablero = Actors.remote().typedActorFor(VistaTablero.class, "vista-tablero", "localhost", 2552);

```

## Quinta implementación, integración con Spring y mecanismo de tolerancia a fallos

Akka tiene un módulo de integración con Spring que permite definir actores directamente en el contexto de Spring. El mecanismo funciona correctamente aunque es un poco complicado de utilizar, y cuesta un poco entender como se utilizan ciertos aspectos.

En esta implementación además de configurar los actores en Spring, se implementó un mecanismo de tolerancia a fallos. La tolerancia a fallos en Akka se implementa mediante Supervisores que reinician los Actores cuando estos lanzen determinadas excepciones, restaurandolos a su estado correcto. La estrategia entonces es dejar fallar a los actores, y que los supervisores los reinicien. Existen varias estrategias de reinicio, en particular porque en algunos casos queremos que se reinicie solo el actor que falló y en otros queremos que se reinicie todo un grupo de actores que trabaja en conjunto.

En este caso, implementamos un mecanismo de envio de mensajes de movimiento a los actores Robot dentro de un actor tipado que falla de manera aleatoria (esto en las implementaciones anteriores se hizo desde un thread aparte con un loop):

``` java MotorMovimiento.java

public interface MotorMovimiento {

  void moverRobots();

  void definirTablero(Tablero tablero);

}

```

``` java MotorMovimientoImpl.java

public class MotorMovimientoImpl extends TypedActor implements MotorMovimiento {
  private Tablero tablero;

  @Override
  public  void moverRobots() {
    for (Robot robot : tablero.getRobots()) {
      robot.moveteEnDireccion(Direccion.direccionAtRandom());
    }
    try {
      Thread.sleep(Math.abs(new Random().nextInt()) % 500);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }
    if(Math.abs(new Random().nextInt()) % 15 == 0){
      throw new RuntimeException("Excepcion aleatoria!");
    }
    ((MotorMovimiento) this.context().getSelf()).moverRobots();
  }

  @Override
  public  void postRestart(Throwable reason) {
    selfSend();
  }
  
  private  void selfSend(){
    ((MotorMovimiento) this.context().getSelf()).moverRobots();
  }

  @Override
    public  void definirTablero(Tablero tablero) {
    this.tablero = tablero;
    }
}

```

En este caso, el actor se envia mensajes a si mismo para funcionar constantemente. Aleatoriamente lanza una RuntimeException, cuando esto sucede, el actor falla y se reinicia. En el reinicio (metodo post-restart) se colocó la condición de que el actor se vuelva a enviar a si mismo el mensaje original. Y se configuró via Spring el supervisor de este actor:

``` xml

<akka:supervision id="motorMovimientoSupervisor">
    <akka:restart-strategy failover="AllForOne"
        retries="10" timerange="1000">
        <akka:trap-exits>
            <akka:trap-exit>java.lang.Exception</akka:trap-exit>
        </akka:trap-exits>
    </akka:restart-strategy>

    <akka:typed-actors>
        <akka:typed-actor id="motorMovimiento"
            interface="com.tenpines.prototypes.akka.movingrobots.actortyped.actores.MotorMovimiento"
            implementation="com.tenpines.prototypes.akka.movingrobots.actortyped.actores.MotorMovimientoImpl"
            lifecycle="permanent" timeout="1000" depends-on="tablero">
        </akka:typed-actor>
    </akka:typed-actors>
</akka:supervision>

```

Lo que se configuró aquí es que se reinicie el actor, con un limite temporal de reintentos (no mas de 10 reintentos en 1000 milisegundos, para prevenir reintentar demasiado en situaciones donde los actores inmediatamente se caen), y ante cualquier excepción (mencionadas en la clausula trap-exit, en este caso aplican todas porque se especificó java.lang.Exception).

Sexta implementación, integración con Apache Camel
Apache Camel es un framework de ruteo de mensajes que funciona con multiples protocolos como HTTP, JMS, SOAP, FTP, y muchisimos más, dentro de una única API basada en URIs. Akka está integrado con este framework permitiendo enviar y recibir mensajes de los actores por medio de estos protocolos, esta es una forma simple de utilizar Akka para interactuar con otros sistemas con protocolos como JMS o Webservices.

En esta implementación, utilizamos un nuevo actor no tipado, VistaTableroJson, que implementa una vista JSON del tablero respondiendo cada mensaje con un String con el tablero serializado. Luego levantar el tablero en una dirección HTTP se realiza mediante la integración de Akka con Apache Camel y la integración de Apache Camel con Jetty (un servidor web liviano escrito en Java):

``` java

final ActorRef vistaTableroJson = (ActorRef) context.getBean("vistaTableroJson");
CamelServiceManager.startCamelService();
CamelContextManager.mandatoryContext().addRoutes(new RouteBuilder(){

@Override
public void configure() throws Exception {
    from("jetty:http://localhost:8080/tablero").to("actor:uuid:"+vistaTableroJson.getUuid() );
}});

```

Al entrar en la url jetty: `http://localhost:8080/tablero`, se puede ver el estado del tablero representado como JSON.

## Conclusiones

Akka permite manejar la concurrencia de una forma distinta, es una herramienta que se puede tener en cuenta para resolver varios tipos de problemas, ya sea utilizando solamente STM, o también actores, actores remotos, la integración con Apache Camel, etc. Hay que tener en cuenta que estas implementaciones de ejemplo son ejemplos y habría que ver que clase de problemas podrían llegar a aparecer en un ambiente productivo. De todas maneras, en la página de Akka hay una [lista de empresas utilizando Akka en producción](http://akka.io/docs/akka/1.1.1/additional/companies-using-akka.html) como referencia de su uso productivo.

Akka, en su proxima versión 2.0 que aún (a febrero 2012) no salió, promete mejorar incluso más las cosas. Se planea agregar mecanismos para load balancing automatico y manejo de la carga en clusters, incluyendo migraciones de actores de una maquina a otra de manera explicita o implicita. Y esto sería por medio de configuración, por lo que el mismo programa serviría para correr en una sola máquina o en varias según se necesite. También incluirá la posibilidad de que se utilice un sistema de eventos con suscriptores, permitiendo que los actores se comuniquen entre sí sin conocerse.

El código fuente de las implementaciones está disponible [aquí](http://dl.dropbox.com/u/2520726/Akka-example-MovingRobots.zip).