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