# Evidencia — Patrón Proxy en Spring Framework (Spring AOP)

**Clase:** `org.springframework.aop.framework.JdkDynamicAopProxy`
**Módulo:** `spring-aop`
**Repositorio:** https://github.com/spring-projects/spring-framework
**Ruta aproximada:** `spring-aop/src/main/java/org/springframework/aop/framework/JdkDynamicAopProxy.java`

## Fragmento representativo (resumido y comentado)

```java
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {

    private final AdvisedSupport advised;

    @Override
    public Object getProxy(@Nullable ClassLoader classLoader) {
        // Crea dinámicamente una clase proxy que implementa las mismas
        // interfaces que el objeto real (el "target"), usando el
        // Proxy de java.lang.reflect y este mismo objeto como InvocationHandler.
        Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
        return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // Antes de invocar el método real, intercepta la llamada y
        // ejecuta la cadena de advices (transacciones, seguridad, logging, etc.)
        MethodInvocation invocation = new ReflectiveMethodInvocation(
                proxy, this.advised.getTargetSource().getTarget(), method, args, ...);
        return invocation.proceed();
    }
}
```

`getProxy` construye un objeto que implementa las mismas interfaces que el bean real, pero cuyas llamadas pasan por `invoke`. Ahí es donde se ejecuta la cadena de *advices* (por ejemplo `@Transactional`, `@Async`, seguridad) antes o después de delegar al objeto real (`target`). El código cliente invoca métodos sobre el proxy exactamente igual que si fuera el objeto real, sin saber que existe intermediación.
