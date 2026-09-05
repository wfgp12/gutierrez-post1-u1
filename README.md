# Post-contenido — Unidad 1: Fundamentos de Patrones de Diseño y Buenas Prácticas

## Descripción
Repositorio del post-contenido de la Unidad 1 de Patrones de Diseño
de Software — Sexto Semestre. Contiene dos partes: refactorización
SOLID de un God Object (parte-1-refactorizacion-solid/) y análisis
de patrones GoF en Spring Framework (parte-2-analisis-gof-spring/).

## Parte 1 — Refactorización SOLID

## Análisis de Violaciones SOLID

| Principio | Método/Sección afectada | Descripción de la violación |
|-----------|--------------------------|------------------------------|
| SRP | calculateTotal + applyDiscount + saveOrder + sendEmail + printReport | La clase concentra cinco responsabilidades sin relación entre sí (cálculo de totales, descuentos, persistencia, notificación y presentación). Un cambio en la forma de enviar el email obliga a tocar una clase que también calcula impuestos y genera reportes, aumentando el riesgo de romper algo que no tiene nada que ver con el cambio solicitado. |
| OCP | applyDiscount (if/else sobre customerType) | El descuento se decide con una cadena de `if/else` sobre un `String`. Agregar un nuevo tipo de cliente exige modificar el método existente en vez de extenderlo, lo que viola "abierto a extensión, cerrado a modificación" y arriesga romper los casos ya probados. |
| DIP | Toda la clase (dependencias internas sin abstracciones) | `OrderProcessor` no depende de ninguna interfaz: gestiona su propia lista en memoria y usa `System.out` como si fuera la base de datos y el servicio de correo. No hay forma de sustituir la persistencia o la notificación sin modificar directamente esta clase. |

Proyecto Maven que refactoriza OrderProcessor aplicando SRP, OCP y
DIP. Ver parte-1-refactorizacion-solid/.

## Parte 2 — Análisis de Patrones GoF en Spring

| # | Patrón | Categoría | Clase en Spring |
|---|--------|-----------|-----------------|
| 1 | Singleton | Creacional | `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry` |
| 2 | Proxy | Estructural | `org.springframework.aop.framework.JdkDynamicAopProxy` |
| 3 | Observer | Comportamiento | `org.springframework.context.ApplicationListener` / `ApplicationEventPublisher` |

Ver parte-2-analisis-gof-spring/documento-analisis.md.

## Herramientas utilizadas
- Java 17, Apache Maven, VS Code, Git, GitHub
- Código fuente de Spring Framework (investigación)

## Conclusiones

La refactorización de `OrderProcessor` mostró en la práctica cómo un God Object concentra responsabilidades que deberían vivir en clases independientes, y cómo aplicar SRP, OCP y DIP reduce el acoplamiento y facilita agregar comportamiento nuevo (como un tipo de descuento) sin modificar código existente. El análisis de Spring Framework confirmó que estos mismos principios sostienen el diseño de un framework maduro: Singleton controla el ciclo de vida de los beans, Proxy permite añadir comportamiento transversal (AOP) sin tocar las clases de negocio, y Observer desacopla a los publicadores de eventos de sus consumidores. La lección principal es que los patrones GoF no son un ejercicio académico aislado, sino la forma concreta en que los principios SOLID se materializan en decisiones de diseño reales.