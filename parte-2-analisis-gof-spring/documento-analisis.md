# Análisis de Patrones de Diseño GoF en Spring Framework

**Nombre:** Gutiérrez
**Curso:** Patrones de Diseño de Software
**Unidad:** Unidad 1 — Post-contenido
**Fecha:** Septiembre de 2026

---

## Introducción

Este documento analiza tres patrones de diseño del catálogo Gang of Four (GoF) presentes en el código fuente de Spring Framework, uno de los frameworks Java más utilizados en la industria para construir aplicaciones empresariales. El objetivo es identificar, en clases concretas del proyecto `spring-projects/spring-framework`, cómo un framework maduro aplica patrones de diseño para resolver problemas reales de gestión del ciclo de vida de objetos, extensión de comportamiento y comunicación desacoplada entre componentes, y conectar cada decisión de diseño con los principios SOLID que la sustentan. Spring Boot, construido sobre Spring Framework, se toma como caso de estudio porque su contenedor de inversión de control (IoC) y su modelo de beans son el punto de entrada más común para cualquier desarrollador que use anotaciones como `@Component`, `@Service` o `@Repository` sin ver el mecanismo interno que las sostiene.

---

## Análisis de Patrón 1 — Singleton

El patrón Singleton pertenece a la categoría **Creacional** y su propósito general es garantizar que una clase tenga una única instancia dentro de un contexto determinado, proporcionando un punto de acceso global controlado a dicha instancia, en lugar de dejar que cualquier parte del sistema cree copias adicionales de un mismo recurso compartido.

En Spring Framework este patrón aparece en `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`, dentro del módulo `spring-beans`. Esta clase es la base de la jerarquía de fábricas de beans (`AbstractBeanFactory` y, en última instancia, `DefaultListableBeanFactory`) que usa cualquier `ApplicationContext` de Spring Boot para instanciar y almacenar sus beans.

El problema específico que resuelve es evitar que un contenedor con potencialmente cientos de beans (servicios, repositorios, controladores) cree una instancia nueva cada vez que un componente los solicita mediante inyección de dependencias. Sin este mecanismo, cada `@Autowired` generaría una instancia distinta del mismo bean, multiplicando el uso de memoria y, más grave aún, rompiendo el estado compartido que muchos servicios asumen (por ejemplo, un pool de conexiones o una caché en memoria). Spring resuelve esto con una tabla de singletons (`singletonObjects`) que se consulta antes de fabricar cualquier bean: si ya existe una instancia para ese nombre, se reutiliza; si no, se crea una sola vez y se registra.

Como evidencia de código, se documentó en `parte-2-analisis-gof-spring/evidencia/singleton-evidence.md` el método `getSingleton(String, ObjectFactory<?>)`, que sincroniza el acceso al mapa de singletons y solo invoca la fábrica del bean (`ObjectFactory.getObject()`) cuando no existe todavía una entrada para ese nombre, guardando el resultado con `addSingleton` para las siguientes solicitudes.

Este patrón refuerza principalmente el **Principio de Responsabilidad Única (SRP)**: `DefaultSingletonBeanRegistry` tiene la única responsabilidad de gestionar el ciclo de vida y la unicidad de las instancias de bean, separada de la responsabilidad de construirlas (que recae en `BeanFactory`/`AbstractAutowireCapableBeanFactory`) o de configurarlas (que recae en los `BeanDefinitionReader`). Si Spring no aplicara este patrón, cada framework o librería que necesitara un recurso compartido tendría que reimplementar su propio mecanismo de instancia única, y el comportamiento de "singleton por contenedor" tendría que gestionarlo manualmente cada desarrollador.

---

## Análisis de Patrón 2 — Proxy

El patrón Proxy pertenece a la categoría **Estructural** y su propósito general es proporcionar un objeto sustituto o intermediario que controla el acceso a otro objeto (el objeto real), permitiendo añadir comportamiento antes o después de delegar la llamada, sin que el código cliente note la diferencia.

En Spring Framework este patrón se encuentra en `org.springframework.aop.framework.JdkDynamicAopProxy`, dentro del módulo `spring-aop`, que es la base de Spring AOP (Aspect-Oriented Programming) y del soporte declarativo de anotaciones como `@Transactional`, `@Async` o `@Cacheable` en Spring Boot.

El problema que resuelve es aplicar comportamiento transversal (transacciones, seguridad, logging, caché) a métodos de negocio sin ensuciar esas clases con código repetido de infraestructura. Una alternativa directa sería que cada método de servicio abriera y cerrara manualmente una transacción o verificara permisos al inicio y al final, duplicando lógica en decenas de clases. Spring AOP resuelve esto envolviendo el bean real en un proxy dinámico: cuando el contenedor detecta que un bean necesita comportamiento adicional (por ejemplo, por tener `@Transactional`), no expone el bean original, sino un proxy que implementa las mismas interfaces.

La evidencia de código, documentada en `parte-2-analisis-gof-spring/evidencia/proxy-evidence.md`, muestra el método `getProxy(ClassLoader)`, que construye el proxy con `java.lang.reflect.Proxy.newProxyInstance(...)` usando el propio `JdkDynamicAopProxy` como `InvocationHandler`, y el método `invoke(Object, Method, Object[])`, donde cada llamada del cliente se intercepta y se ejecuta a través de una cadena de *advices* (`MethodInvocation`) antes de delegar, si corresponde, al objeto real (`target`).

Este patrón refuerza directamente el **Principio de Abierto/Cerrado (OCP)**: es posible añadir comportamiento nuevo a un bean existente (una transacción, una traza de auditoría) simplemente agregando una anotación o un *advice*, sin modificar ni una línea del código fuente de la clase de negocio. También refuerza el SRP, porque separa la lógica de negocio (en el bean real) de la lógica transversal (en los *advices* ejecutados por el proxy). Sin este patrón, Spring Boot no podría ofrecer `@Transactional` como una simple anotación: cada equipo tendría que escribir manualmente el código de apertura/cierre de transacción alrededor de cada método sensible.

---

## Análisis de Patrón 3 — Observer

El patrón Observer pertenece a la categoría **Comportamiento** y su propósito general es definir una dependencia de uno-a-muchos entre objetos, de modo que cuando un objeto (el sujeto) cambia de estado o produce un evento, todos sus dependientes (los observadores) sean notificados automáticamente, sin que el sujeto necesite conocer los detalles de cada observador.

En Spring Framework este patrón corresponde al mecanismo de eventos de la aplicación, implementado principalmente en `org.springframework.context.ApplicationListener` (el rol de observador), `org.springframework.context.ApplicationEventPublisher` (el rol de sujeto, expuesto a través de `ApplicationContext`) y `org.springframework.context.event.SimpleApplicationEventMulticaster` (el despachador de eventos), todos dentro del módulo `spring-context`.

El problema que resuelve es permitir que distintas partes de una aplicación Spring Boot reaccionen a un mismo suceso (por ejemplo, "la aplicación terminó de arrancar" o "se registró un nuevo usuario") sin que el componente que dispara el evento tenga que conocer ni invocar directamente a cada uno de los interesados. La alternativa directa —que el componente emisor llame explícitamente a cada método interesado— acoplaría fuertemente módulos que deberían ser independientes (por ejemplo, el módulo de registro de usuarios tendría que conocer y llamar al módulo de envío de correos, al de auditoría y al de métricas).

Como evidencia de código, documentada en `parte-2-analisis-gof-spring/evidencia/observer-evidence.md`, se muestra la interfaz `ApplicationListener<E extends ApplicationEvent>` con su único método `onApplicationEvent(E event)`, y el método `multicastEvent(ApplicationEvent, ResolvableType)` de `SimpleApplicationEventMulticaster`, que recorre todos los *listeners* registrados que aplican a un evento y les notifica invocando ese método, de forma síncrona o a través de un `Executor` si se configuró procesamiento asíncrono.

Este patrón refuerza principalmente el **Principio de Inversión de Dependencias (DIP)** y, de forma secundaria, el OCP: el publicador del evento (`ApplicationEventPublisher`) y los observadores (`ApplicationListener`) dependen únicamente de la abstracción `ApplicationEvent`/`ApplicationListener`, no el uno del otro; y es posible agregar un nuevo observador a un evento existente simplemente registrando un nuevo bean `ApplicationListener` o un método anotado con `@EventListener`, sin modificar el código que publica el evento. Sin este patrón, cada nueva funcionalidad que necesitara reaccionar a un evento del ciclo de vida de la aplicación obligaría a modificar el componente que lo origina.

---

## Conclusiones

El análisis de estos tres patrones confirma que Spring Framework no aplica patrones de diseño como ejercicio decorativo, sino como solución directa a problemas concretos de su dominio: Singleton controla el ciclo de vida y la unicidad de los beans dentro del contenedor IoC, Proxy permite inyectar comportamiento transversal (transacciones, seguridad, caché) sin modificar las clases de negocio, y Observer desacopla completamente a los emisores de eventos de sus consumidores. En los tres casos, el patrón está atado a uno o más principios SOLID —SRP en Singleton, OCP en Proxy, DIP en Observer— lo que demuestra que los patrones GoF son, en la práctica, la forma concreta en que esos principios se materializan en decisiones de diseño verificables en código real. La lección para el diseño propio es que un patrón no debería elegirse por moda, sino porque resuelve un problema de acoplamiento o de extensión que ya se identificó explícitamente, tal como ocurre en la refactorización de `OrderProcessor` realizada en la Parte 1 de esta actividad.

---

## Referencias

- Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.
- VMware Tanzu / Spring Team. (2024). *Spring Framework Documentation — Core Technologies*. https://docs.spring.io/spring-framework/reference/core.html
- Spring Team. (2024). *Spring Boot Reference Documentation*. https://docs.spring.io/spring-boot/reference/
- Refactoring Guru. (2024). *Design Patterns*. https://refactoring.guru/design-patterns
- Spring Projects. (2024). *spring-framework* [Repositorio de código fuente]. GitHub. https://github.com/spring-projects/spring-framework
