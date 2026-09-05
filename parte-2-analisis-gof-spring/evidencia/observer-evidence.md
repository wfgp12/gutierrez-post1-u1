# Evidencia — Patrón Observer en Spring Framework (Spring Events)

**Interfaces/Clases:** `org.springframework.context.ApplicationListener`, `org.springframework.context.ApplicationEventPublisher`, `org.springframework.context.event.SimpleApplicationEventMulticaster`
**Módulo:** `spring-context`
**Repositorio:** https://github.com/spring-projects/spring-framework
**Ruta aproximada:** `spring-context/src/main/java/org/springframework/context/event/SimpleApplicationEventMulticaster.java`

## Fragmento representativo (resumido y comentado)

```java
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    // Método que cada "observador" implementa para reaccionar al evento
    void onApplicationEvent(E event);
}

public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster {

    @Override
    public void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType) {
        ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
        // Recorre todos los listeners registrados que aplican a este tipo de evento
        for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
            Executor executor = getTaskExecutor();
            if (executor != null) {
                executor.execute(() -> invokeListener(listener, event));
            } else {
                invokeListener(listener, event); // notifica al observador
            }
        }
    }
}
```

`ApplicationListener` es el rol de "Observador" del patrón: cualquier bean que lo implemente (o use `@EventListener`) se registra ante el contenedor. `ApplicationEventPublisher.publishEvent(...)` (el "Sujeto") no conoce a sus observadores concretos; delega en `SimpleApplicationEventMulticaster`, que recorre la lista de listeners registrados y llama a `onApplicationEvent` en cada uno. El publicador y los observadores quedan completamente desacoplados entre sí.
