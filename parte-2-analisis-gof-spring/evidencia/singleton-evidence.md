# Evidencia — Patrón Singleton en Spring Framework

**Clase:** `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry`
**Módulo:** `spring-beans`
**Repositorio:** https://github.com/spring-projects/spring-framework
**Ruta aproximada:** `spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java`

## Fragmento representativo (resumido y comentado)

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry
        implements SingletonBeanRegistry {

    /** Cache de singletons: nombre de bean -> instancia del bean. */
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

    public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        synchronized (this.singletonObjects) {
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                // Solo se crea la instancia si aún no existe en la cache
                singletonObject = singletonFactory.getObject();
                addSingleton(beanName, singletonObject);
            }
            return singletonObject;
        }
    }

    protected void addSingleton(String beanName, Object singletonObject) {
        synchronized (this.singletonObjects) {
            this.singletonObjects.put(beanName, singletonObject);
        }
    }
}
```

`getSingleton` consulta primero el mapa `singletonObjects`; solo invoca la `ObjectFactory` (que termina llamando al constructor del bean) cuando no hay una instancia registrada todavía, y guarda el resultado para las siguientes solicitudes. Este es el mecanismo real que hace que un bean con scope por defecto (`singleton`) se instancie una única vez por contenedor `ApplicationContext`.
