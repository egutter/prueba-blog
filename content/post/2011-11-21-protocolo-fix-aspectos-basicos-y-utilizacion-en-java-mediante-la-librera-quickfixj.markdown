---
author: Nicolas Perez Santoro
categories:
- java
- finantial
- library
comments: true
date: 2011-11-21T00:00:00Z
title: 'protocolo fix: aspectos básicos y utilización en java mediante la librería
  quickfix/j'
url: /2011/11/21/protocolo-fix-aspectos-basicos-y-utilizacion-en-java-mediante-la-librera-quickfixj/
---

## Introducción

El protocolo **FIX ** (*Financial Information Exchange Protocol*) es un protocolo de mensajes para el comercio de instrumentos financieros. **FIX** se utiliza ampliamente para la comunicación automática entre los participantes del intercambio de instrumentos, y especifica como son los mensajes para crear ordenes de compra y venta y consultar cotizaciones de instrumento, entre otros. Este protocolo es el que hay que utilizar para comunicarnos con practicamente todos los mercados financieros de manera electrónica.

En este articulo, veremos los aspectos básicos del protocolo **FIX**, como utilizarlo mediante la librería de java **QuickFIX/J** mediante un ejemplo como cliente y servidor, y algunos aspectos poco obvios que hay que tener en cuenta al utilizarlo.

<!--more-->

## Diferentes versiones del protocolo FIX

El protocolo tiene diferentes versiones, el mismo fue evolucionando a través de los años, agregando diferentes mensajes y campos para acomodar las funcionalidades requeridas por los interlocutores de todo el mundo. La versión del protocolo se divide en dos: la versión del protocolo de mensajes en si (por ejemplo, **FIX 5.0 SP2**), y la versión del protocolo de transporte de **FIX**, **FIXT** (por ejemplo, **FIXT 1.1**). Las versiones anteriores a la 5 no tenían esa division de protocolo de mensajes y protocolo de transporte, y se denotan únicamente por el número de version del protocolo.

El protocolo **FIX** soporta extensiones, agregando nuevos mensajes y campos dentro de mensajes existentes, definiendo nuevos códigos de mensaje y nuevos números de campos. Para esto, los interlocutores deben conocer y estar de acuerdo en estas extensiones. De todas maneras, el protocolo **FIX** tiene una gran cantidad de campos pensados para muchisimos casos de usos, ampliandose versión a versión, por lo que la necesidad de extender el protocolo debería en teoría ser muy baja. Tampoco es necesario implementar la totalidad de los mensajes y todos los códigos opcionales existentes, sino solo los que tengan sentido y sean utilizados por nuestros interlocutores.

## Cómo es el formato FIX

El formato **FIX** es un formato textual, donde cada mensaje es una única linea de la forma:

`NUMERO_DE_CAMPO=CONTENIDO[SEPARADOR]NUMERO_DE_CAMPO=CONTENIDO[SEPARADOR]....`

El separador es el caracter unicode representado en Java y otros como **"\u0001"**. Los campos se identifican por su número, y el tipo de mensaje es un campo más dentro del mensaje (el campo 35), y el último campo es el campo 10 que es el checksum. El orden de los campos importa, y está especificado para cada mensaje **FIX**. Las estructuras repetitivas en **FIX** se llaman **Repeating Group** y se definen simplemente definiendo un campo cabecera del **Repeating Group** que indica la cantidad de elementos contenidos, y todos los elementos se diferencian entre sí detectando el primer campo de cada elemento, esto funciona precisamente porque importa el orden de los campos.

## QuickFIX/J

**FIX** es un protocolo abierto y gratuito; es solo la especificación de los mensajes y de los campos de los mismos, y existen varias implementaciones de este protocolo. **QuickFIX/J** es una implementación gratuita y open source (con una licencia propia similar a **BSD**/**MIT**) para Java (es un port de **QuickFIX**, que está escrita en C++), que es muy fácil de utilizar para comunicarnos con cualquier participante, ya sea como cliente o servidor.

*Nota: QuickFIX/J no implementa lógica de negocio alguna, es decir, no implementa ningún motor de cotizaciones, matcheo de ordenes de compra y venta, etc; lo que implementa es únicamente es el protocolo de comunicación.*

## Ejemplos de Cliente y Servidor en QuickFIX/J

Vamos a hacer un cliente de ejemplo en **QuickFIX/J** versión 1.5.1. Este cliente se conectará a un servidor, y apenas se conecte enviará una orden de compra de un instrumento ficticio, el servidor responderá con otro mensaje, y cuando el cliente reciba la respuesta de la compra, seteará una variable, y observaremos la respuesta de esa variable. Con este ejemplo, si bien irreal, veremos como se utiliza **QuickFIX/J** como cliente y como servidor.

El primer paso es [bajar la librería](http://www.quickfixj.org/downloads/). Hay varios Jars, uno por cada versión del protocolo **FIX**, pero vamos a incluir simplemente el que tiene todo ("all"), en este caso voy a incluir la última versión: **quickfixj-all-1.5.1.jar**. También debemos bajar (o bien configurar via Maven) las dependencias especificadas [aquí](http://www.quickfixj.org/quickfixj/usermanual/1.5.1/installation.html#dependencies).

Incluida esta librería en nuestro proyecto, tenemos que implementar nuestro cliente, y para esto **QuickFIX/J** nos pide que implementemos la interfaz `quickfix.Application`. **QuickFIX/J** es un framework que trabaja con el concepto de *Inversión de Control*: en lugar de invocar los métodos de **QuickFIX/J** para loguearnos, ir recibiendo los mensajes, etc; nosotros tenemos que darle un objeto `Application` y **QuickFIX/J** invoca a los métodos de ese objeto con los eventos que vayan ocurriendo, en sus propios threads.

La interfaz `Application` define métodos para cada evento que ocurra durante la comunicación, por ejemplo `onLogon`, `onLogout`, `fromApp` (se recibió un mensaje de negocio), `fromAdmin` (se recibio un mensaje relativo a la sesión, como por ejemplo rechazo de un mensaje mal formado), entre otros. Nosotros, en vez de implementar `Application` directamente, vamos a heredar de `ApplicationAdapter` que es una clase abstracta que tiene definiciones vacías para todos estos métodos por default. Es la misma interfaz tanto para Cliente como para Servidor.

``` java ExampleClientApplication.java

package com.tenpines.ejemplofix;

import quickfix.ApplicationAdapter;
import quickfix.FieldNotFound;
import quickfix.Message;
import quickfix.Session;
import quickfix.SessionID;
import quickfix.SessionNotFound;
import quickfix.field.ExecType;
import quickfix.field.Password;
import quickfix.field.Username;
import quickfix.fix50.ExecutionReport;
import quickfix.fix50.NewOrderSingle;
import quickfix.fixt11.Logon;

public class ExampleClientApplication extends ApplicationAdapter {
  private boolean seEjecutoOrdenCorrectamente = false;
  private  boolean estaLogueado = false;

  private final NewOrderSingle newOrder;
  private final String usuario;
  private final String password;

  public ExampleClientApplication(NewOrderSingle newOrder, String usuario, String password) {
    this.newOrder = newOrder;
    this.usuario = usuario;
    this.password = password;
  }

  @Override
  public  void onLogon(SessionID sessionId) {
    this.estaLogueado = true;
    try {
      Session.sendToTarget(this.newOrder, sessionId);
    } catch (SessionNotFound e) {
      throw new RuntimeException(e);
    }
  }

  @Override
  public  void fromApp(Message message, SessionID sessionId) throws FieldNotFound {
    if (message instanceof ExecutionReport) {
      ExecutionReport executionReport = (ExecutionReport) message;
      if (esExecutionReportNew(executionReport)) {
        // Como el mensaje corresponde a una creacion de orden,
        // verifico que se refiera a la misma orden que acabo de enviar
        // comparando su ClOrdID.
        if (esClOrdIDCorrecto(executionReport)) {
          this.seEjecutoOrdenCorrectamente = true;
        }
      }
    }
  }

  private  boolean esClOrdIDCorrecto(ExecutionReport executionReport) throws FieldNotFound {
    return executionReport.getClOrdID().getValue().equals(this.newOrder.getClOrdID().getValue());
  }

  private  boolean esExecutionReportNew(ExecutionReport executionReport) throws FieldNotFound {
    return executionReport.getExecType().getValue() == ExecType.NEW;
  }

  @Override
  public  void toAdmin(Message message, SessionID sessionId) {
    if (message instanceof Logon) {
      message.setField(new Username(usuario));
      message.setField(new Password(password));
    }
  }

  public  boolean estaLogueado() {
    return estaLogueado;
  }

  public  boolean seEjecutoOrdenCorrectamente() {
    return seEjecutoOrdenCorrectamente;
  }
}

```

Esta clase en su constructor recibirá el mensaje `NewOrderSingle` de creación de una orden, que el cliente deberá enviar al loguearse. En el método `onLogon`, el cual **QuickFIX/J** invoca luego de un logueo exitoso, realiza el envío del mensaje con el método estático `Session.sendToTarget`.

En el método `fromApp`, se reciben los mensajes de negocio, y lo que buscamos es un mensaje de `ExecutionReport`, que es el mensaje de **FIX** con el que se reportan los cambios de estado de una orden (cuando es creada, cuando es cancelada, a medida que se va ejecutando, cuando es completada, etc).

Si el campo `ClOrdID` (el cual representa el *id de orden del cliente*) del `ExecutionReport` coincide con el que mandamos en el `NewOrderSingle`, seteamos el flag de `seEjecutoOrdenCorrectamente`.

Finalmente en el método `toAdmin`, que se invoca inmediatamente antes de enviar un mensaje "administrativo" (es decir, un mensaje que no es de negocio), interceptamos el mensaje `Logon` (que se crea automáticamente) y le asignamos usuario y password (sí, esa es la forma estándar de definir el usuario y password de la conexión).

``` java ExampleServerApplication.java

package com.tenpines.ejemplofix;

import quickfix.ApplicationAdapter;
import quickfix.FieldNotFound;
import quickfix.IncorrectTagValue;
import quickfix.Message;
import quickfix.RejectLogon;
import quickfix.Session;
import quickfix.SessionID;
import quickfix.SessionNotFound;
import quickfix.UnsupportedMessageType;
import quickfix.field.ClOrdID;
import quickfix.field.CumQty;
import quickfix.field.ExecID;
import quickfix.field.ExecType;
import quickfix.field.LeavesQty;
import quickfix.field.OrdStatus;
import quickfix.field.OrderID;
import quickfix.fix50.ExecutionReport;
import quickfix.fix50.NewOrderSingle;
import quickfix.fixt11.Logon;

public class ExampleServerApplication extends ApplicationAdapter {

  @Override
  public  void fromAdmin(Message message, SessionID sessionId) throws FieldNotFound, RejectLogon {
    if (message instanceof Logon) {
      if (!usuarioYPasswordCorrectos((Logon) message)) {
        throw new RejectLogon();
      }
    }
  }

  private  boolean usuarioYPasswordCorrectos(Logon logon) throws FieldNotFound {
    return logon.getUsername().getValue().equals("usuario") 
        && logon.getPassword().getValue().equals("password");
  }

  @Override
  public  void fromApp(Message message, SessionID sessionId) throws FieldNotFound, IncorrectTagValue,
          UnsupportedMessageType {
    if (message instanceof NewOrderSingle) {
      NewOrderSingle newOrderSingle = ((NewOrderSingle) message);
      ExecutionReport executionReport = new ExecutionReport();
      String clOrdID = newOrderSingle.getClOrdID().getValue();
      executionReport.set(new ClOrdID(clOrdID));
      executionReport.set(new ExecID("98765"));
      executionReport.set(new OrderID("99999"));
      executionReport.set(newOrderSingle.getSide());
      executionReport.set(new OrdStatus(OrdStatus.NEW));
      executionReport.set(new CumQty(0));
      executionReport.set(new ExecType(ExecType.NEW));
      executionReport.set(new LeavesQty(newOrderSingle.getOrderQty().getValue()));
      try {
        Session.sendToTarget(executionReport, sessionId);
      } catch (SessionNotFound e) {
        throw new RuntimeException(e);
      }
    }
  }

}

```

Este es el `Application` del servidor de ejemplo. En el método `fromAdmin` (donde se reciben los mensajes administrativos), validamos el `Login`, si no coincide el usuario y password, lanzamos la excepcion `RejectLogin`.

**Nota:** en este esquema sencillo, el password viaja como texto plano: Las conexiones de FIX tienen que segurizarse por medio de algún esquema como SSL, utilizar una VPN, o similar.

En el metodo `fromApp` recibimos el `NewOrderSingle` y creamos un mensaje de `ExecutionReport` y le asignamos varios campos (en este ejemplo, solo los obligatorios). En el campo `ClOrdID`, copiamos el dato recibido por el `NewOrderSingle`, para que el cliente pueda detectar a que `ExecutionReport` corresponde. Y finalmente enviamos el mensaje.

``` java ExampleFixTest.java

package com.tenpines.ejemplofix;

import java.util.Date;

import org.junit.Assert;
import org.junit.Test;

import com.tenpines.ejemplofix.ExampleClientApplication;
import com.tenpines.ejemplofix.ExampleServerApplication;

import quickfix.Application;
import quickfix.ConfigError;
import quickfix.DefaultMessageFactory;
import quickfix.MemoryStoreFactory;
import quickfix.ScreenLogFactory;
import quickfix.SessionSettings;
import quickfix.SocketAcceptor;
import quickfix.SocketInitiator;
import quickfix.field.ClOrdID;
import quickfix.field.OrdType;
import quickfix.field.OrderQty;
import quickfix.field.Side;
import quickfix.field.TransactTime;
import quickfix.fix50.NewOrderSingle;

public class ExampleFixTest {

  @Test
  public  void test() throws Exception {
    iniciarServidor();
    String password = "password";
    String usuario = "usuario";
    NewOrderSingle newOrder = new NewOrderSingle(new ClOrdID("12345"), new Side(Side.BUY), 
        new TransactTime(new Date()), new OrdType(OrdType.MARKET));
    newOrder.set(new OrderQty(1000));
    ExampleClientApplication application = new ExampleClientApplication(newOrder, usuario, password);
    iniciarCliente(application);
    Thread.sleep(5000L); // 5 segundos
    Assert.assertTrue(application.estaLogueado());
    Assert.assertTrue(application.seEjecutoOrdenCorrectamente());
  }

  private  void iniciarCliente(Application application) throws ConfigError {
    SessionSettings settings = new SessionSettings(this.getClass().getResourceAsStream("cliente.cfg"));
    SocketInitiator socketInitiator = new SocketInitiator(application, new MemoryStoreFactory(), settings,
            new ScreenLogFactory(), new DefaultMessageFactory());
    socketInitiator.start();
  }

  private  void iniciarServidor() throws ConfigError {
    SessionSettings settings = new SessionSettings(this.getClass().getResourceAsStream("servidor.cfg"));
    ExampleServerApplication application = new ExampleServerApplication();
    SocketAcceptor acceptor = new SocketAcceptor(application, new MemoryStoreFactory(), settings,
            new ScreenLogFactory(), new DefaultMessageFactory());
    acceptor.start();
  }
}

```

En este test inicializamos el cliente y el servidor, y vemos que efectivamente el cliente se loguea contra el servidor, envia la orden, y recibe la respuesta. Notese que el cliente y el servidor se construyen de maneras distintas pero similares: el primero con un `SocketIniciator` y el segundo con un `SocketAcceptor`. En el constructor de cada uno de ellos se pasan objetos que permiten parametrizar el comportamiento del motor de **QuickFIX/J**, definiendo el esquema de logging, como se guardan los mensajes, y cual es el Factory de los mensajes (que depende de la versión del protocolo de **FIX**, pero `DefaultMessageFactory` sirve para cualquier versión).

Hay que tener cuidado al hacer tests integrales con **QuickFIX/J**, ya que como el `Application` corre en un thread aparte, instanciado por **QuickFIX/J**, perdemos el control de la ejecución y necesitamos algún tipo de espera o sincronismo para asegurarnos de que cumplieron las acciones deseadas (en este caso, que se loguee el cliente contra el servidor y se envien entre ellos los mensajes `NewOrderSingle` y `ExecutionReport`). Por esto hay en el test un `Thread.sleep` con 5 segundos, si este fuera un test real, lo correcto sería utilizar polling con timeout, no `Thread.sleep` ya que es un método obviamente lento.

``` linux-config cliente.cfg

[default]
ConnectionType=initiator
StartTime=00:00:00
EndTime=00:00:00
HeartBtInt=300

[session]
BeginString=FIXT.1.1
DefaultApplVerID=FIX.5.0
SocketConnectHost=localhost
SocketConnectPort=9876
SenderCompID=CLIENTE
TargetCompID=SERVIDOR

```

``` linux-config servidor.cfg

[default]
ConnectionType=acceptor

StartTime=00:00:00
EndTime=00:00:00

HeartBtInt=300
RejectInvalidMessage=N

[session]
BeginString=FIXT.1.1
DefaultApplVerID=FIX.5.0
SocketAcceptPort=9876
SenderCompID=SERVIDOR
TargetCompID=CLIENTE

```

`ConnectionType` identifica el tipo de conexión, es decir, si es servidor o cliente. `StartTime` y `EndTime` indican el inicio y fin de la sesión y está relacionado al tiempo de apertura y cierre de los mercados. `BeginString` indica la versión del protocolo de transporte de **FIX**, y `DefaultApplVerID` indica la versión del protocolo (las versiones anterior a la 5, no tienen un protocolo de transporte especifico diferenciado de la versión de **FIX** en sí, y se definen únicamente por el `BeginString` indicando el protocolo de **FIX**).

`SenderCompID` y `TargetCompID` son identificadores de los interlocutores de la sesión, y están invertidos en el Cliente y el Servidor. En **FIX**, la sesión es algo que se considera que va más allá de la conexión dada en un momento, si la conexión se cae y después se vuelve a conectar, la sesión sigue siendo la misma, y de hecho se reenvían los mensajes pendientes al momento de caerse la sesión (en **QuickFIX/J** para que esto funcione hay que instanciar el `SocketAcceptor/Initiator` con el `FileStoreFactory`, no el `MemoryStoreFactory`, el cual guarda los mensajes en un archivo para poder reenviarlos). Justamente porque la sesión es algo global, el `SessionID` de **QuickFIX/J** está definido básicamente por tres valores: `SenderCompID`, `TargetCompID` y `BeginString`.

## Cuestiones a considerar de QuickFIX/J

**QuickFIX/J** utiliza "diccionarios" que definen los mensajes de **FIX**:

- Los tipos de mensajes y su código identificatorio
- Los campos que contiene cada mensaje, su tipo, valores válidos
- Los repeating groups (cual es el encabezado del grupo y los campos de los elementos del repeating group)

Cada diccionario es un XML que se puede editar. Si abrimos los fuentes de **QuickFIX/J** veremos que tiene un diccionario de datos por cada versión del protocolo **FIX**, y utiliza generación de código en el build para crear todas las clases que representan cada mensaje y cada campo (ya que en **QuickFIX/J**, cada campo es una clase). Si queremos, podemos modificar este XML y configurar **QuickFIX/J** para que utilice nuestro XML, esto es necesario si utilizamos extensiones del protocolo que agreguen nuevos campos.

Justamente por ser estas clases autogeneradas, tienen ciertos aspectos contra-intutivos:

Cada campo está representado por una clase. El campo `OrderQty`, que representa la cantidad a comerciar en una órden y es un número, por ejemplo, está representado por la clase `OrderQty`, y una instancia de `OrderQty` tiene métodos como `getValue` para obtener el valor del campo y `getField` para obtener el número de campo en el protocolo.

En las clases que representan los mensajes, no existen métodos `setCAMPO(tipoDeDato)`, sino unicamente overloads del metodo set que reciben la clase que representa el tipo de dato. Por eso, para setear por ejemplo el campo `OrderQty`, hay que hacer cosas como `newOrder.set(new OrderQty(1000))`. Y los métodos get, si bien están definidos, devuelven el valor envuelto en la clase que representa su campo.

Los **Repeating Groups** también están representados por una clase, y la clase se llama igual que el campo del encabezado. Por ejemplo, en el mensaje `SecurityList`, que contiene una lista de instrumentos financieros, hay un repeating group de instrumentos, y para extraerlos, hay que hacer lo siguiente:

``` java

quickfix.fix50.SecurityList.NoRelatedSym noRelatedSym = new quickfix.fix50.SecurityList.NoRelatedSym();
message.getGroup(indiceDeGrupo + 1, noRelatedSym); // +1 porque el primer grupo tiene indice 1, no 0.
//ahora noRelatedSym tiene los datos del grupo

```

Esto es confuso porque `NoRelatedSym` es en realidad el nombre del campo que tiene la cantidad de simbolos relacionados ("No" en "NoRelatedSym" es una abreviación de "número"), y no del **Repeating Group** en sí, esto es una consecuencia de la generación de código automática de **QuickFix/J**. Para añadir confusión, hay multiples clases llamadas `NoRelatedSym`, cada una de ellas representando un **Repeating Group** distinto dentro de cada mensaje. Por eso estas clases son internas al mensaje (como se puede ver en el ejemplo, esta clase `NoRelatedSym` es interna a la clase `SecurityList`).

### Recursos útiles

- http://www.fixprotocol.org: Donde se puede conseguir la especificación del protocolo en todas sus versiones. Dentro de este sitio se encuentra [FIXimate](http://www.fixprotocol.org/FIXimate3.0/), herramienta que permite consultar mensajes o campos por su código, esencial para entender mensajes de errores y logs de mensajes.
- http://www.quickfixj.org/: Sitio web de QuickFIX/J, con la documentación de la librería.