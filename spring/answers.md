# Spring answers

<details>
    <summary>
        <strong>
            1. Como você valida duas condições em um arquivo YAML ao criar um bean?
        </strong>
    </summary>

Para validações simples, você pode usar a propriedade `@ConditionalOnProperty` em um bean. Por exemplo:

```java
@Configuration
public class AppConfig {

    @Bean
    @ConditionalOnProperty(name = "app.feature.enabled", havingValue = "true")
    public MyService myService() {
        return new MyService();
    }
}
```

Para validações mais complexas, há duas abordagens, usando a anotação `@Conditional` ou `@ConditionalOnExpression`.

Com a anotação `@Conditional`, você pode criar uma classe de condição personalizada que implementa a interface `Condition`. Aqui está um exemplo:

```java
public class MyCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String featureEnabled = context.getEnvironment().getProperty("app.feature.enabled");
        String environment = context.getEnvironment().getProperty("app.environment");
        return "true".equals(featureEnabled) && "production".equals(environment);
    }
}

@Configuration
public class AppConfig {

    @Bean
    @Conditional(MyCustomCondition.class)
    public MyService myService() {
        return new MyService();
    }
}
```

Com a anotação `@ConditionalOnExpression`, você pode usar uma expressão SpEL (Spring Expression Language) para validar várias condições. Aqui está um exemplo:

```java
@Configuration
public class MyConfiguration {

    @Bean
    @ConditionalOnExpression(
        "'${app.feature.enabled}' == 'true' and '${app.environment}' == 'production'"
    )
    public MyBean myBean() {
        return new MyBean();
    }
}
```

</details>

<details>
    <summary>
        <strong>
            2. Como você diagnosticaria e resolveria problemas de desempenho em um aplicativo Spring Boot de alta carga?
        </strong>
    </summary>

### 1. Diagnóstico

- Use ferramentas como **Prometheus + Grafana** ou **Spring Boot Actuator** para monitorar métricas de CPU, memória, uso de threads e latência.
- Verifique logs usando agregadores como **ELK Stack (Elasticsearch, Logstash, Kibana)**.
- Use métricas do Actuator para identificar endpoints lentos ou com muitas requisições.
- **Profiling**: Use ferramentas como VisualVM, JProfiler ou YourKit para identificar consumo excessivo de CPU, bloqueios de threads e vazamentos de memória.
- **Testes de carga**: Utilize JMeter ou Gatling para simular cenários de alta carga e medir o desempenho de componentes críticos.

### 2. Resolução

- Banco de Dados:

  - Configure o cache de segundo nível no Hibernate para reduzir consultas repetidas.
  - Identifique queries lentas e otimize-as, adicionando índices ou refatorando consultas complexas.

- Threads:

  - Ajuste o pool de threads conforme necessário.
  - Habilite o uso de virtual threads.

- Compressão de Respostas:

  - Ative a compressão de respostas para economizar largura de banda.

    ```yaml
    server:
    compression:
      enabled: true
      mime-types: application/json,application/xml,text/html
      min-response-size: 2048
    ```

- Cache:

  - Use Spring Cache com Redis ou Caffeine para reduzir o processamento repetitivo:

    ```java
    @Cacheable("items")
    public List<Item> getItems() {
        return repository.findAll();
    }
    ```

- Rate Limiting:

  - Adote bibliotecas como Resilience4j ou Bucket4j para limitar o tráfego de requisições.

- Servidor:
  - Use inicialização preguiçosa para reduzir o tempo de inicialização e adiar a criação de beans não essenciais.

### 3. Boas Práticas

- **Escalabilidade horizontal**: Utilize balanceadores de carga, como NGINX ou HAProxy, para distribuir o tráfego entre múltiplas instâncias.
- **Prevenção**: Realize testes de carga regularmente e implemente alertas em métricas críticas.
- **Desative recursos desnecessários**: Certifique-se de que autoconfigurações e beans não utilizados estão desativados.

</details>

<details>
    <summary>
        <strong>
            3. Quais etapas você seguiria para dimensionar um aplicativo Spring Boot para aumentar o tráfego?
        </strong>
    </summary>

### 1. Otimização da Aplicação

- Perfis de Configuração:

  - Configure perfis no application.yml para separar ambientes (ex.: dev, prod) e garantir que a aplicação use apenas recursos necessários em produção.

- Redução de Sobrecarga:
  - Desative autoconfigurações desnecessárias adicionando dependências seletivamente
  - Utilize o Spring Boot Actuator para identificar beans não utilizados e otimize o contexto.

### 2. Uso Eficiente de Recursos

- Banco de Dados:

  - Ative caches (ex.: Caffeine, Redis) para reduzir consultas repetitivas.
  - Use conexões de banco otimizadas configurando corretamente o pool (ex.: HikariCP):

    ```yaml
    spring.datasource.hikari:
      maximum-pool-size: 50
      minimum-idle: 10
      idle-timeout: 30000
      max-lifetime: 1800000
    ```

- Thread Pool:

  - Ajuste o thread pool do Tomcat ou Jetty para lidar com mais requisições simultâneas.

    ```yaml
    server:
      tomcat:
      threads:
      max: 200
      min-spare: 20
    ```

### 3. Escalabilidade Horizontal

- Balanceamento de Carga:

  - Configure um balanceador de carga como **NGINX** ou **HAProxy** para distribuir requisições entre várias instâncias.
  - Use ferramentas de escalonamento automático (ex.: AWS Auto Scaling, Kubernetes HPA) para adicionar instâncias dinamicamente.

- Configuração de Statelessness:
  - Torne a aplicação sem estado.

### 4. Testes e Simulação

- Testes de Carga:

  - Utilize **JMeter** ou **Gatling** para simular cenários de alto tráfego e identificar gargalos.

- Ambiente de Testes:
  - Replique o ambiente de produção em uma infraestrutura de teste para validar alterações.

### 5. Uso de CDN

- Configure uma CDN (ex.: Cloudflare, AWS CloudFront) para entregar conteúdo estático e reduzir a carga nos servidores de backend.

### 6. Monitoramento e Métricas

- Implemente o **Spring Boot Actuator** para expor métricas e monitorar uso de recursos (CPU, memória, latência).
- Integre com **Prometheus e Grafana** para configurar alertas automáticos e painéis de monitoramento.

### 7. Cache e Rate Limiting

- Cache Local e Distribuído:

  - Adote caches distribuídos, como **Redis** ou **Hazelcast**, para armazenar dados frequentemente acessados.

- Limitação de Taxa:
  - Proteja a aplicação de sobrecarga usando bibliotecas como Resilience4j para limitar requisições por usuário ou IP.

### 8. Escalabilidade com Containers

- Use **Docker** para empacotar sua aplicação e **Kubernetes** ou **Docker Swarm** para gerenciar a escalabilidade de forma eficiente.

</details>

<details>
    <summary>
        <strong>
            4. Como funcionam as transações com @Transactional e o que acontece internamente?
        </strong>
    </summary>

A anotação `@Transactional` do Spring é usada para gerenciar transações de banco de dados de forma declarativa. Ela garante que um conjunto de operações sejam tratadas como uma única unidade de trabalho (commit ou rollback). Aqui está o funcionamento interno:

---

**1. Interceptação com Proxy**  
Quando você anota um método com `@Transactional`, o Spring cria um proxy dinâmico ao redor do bean, interceptando as chamadas ao método. Esse proxy gerencia o comportamento transacional:

- No caso de **interface**, o proxy é do tipo JDK.
- Para classes concretas, o proxy é baseado em **CGLIB** (subclasse da classe-alvo).

---

**2. Início da Transação**

- Quando o método é invocado, o proxy verifica se já existe uma transação ativa.

  - **Se não existir**: O Spring inicia uma nova transação (geralmente com o `DataSourceTransactionManager` ou `JpaTransactionManager`).
  - **Se existir**: O comportamento depende da política de propagação (ex.: `REQUIRED`, `REQUIRES_NEW`).

- O contexto transacional é associado ao **ThreadLocal**, garantindo isolamento da transação para o thread atual.

---

**3. Execução do Método**

- O método é executado dentro do contexto transacional.
- O **EntityManager** (ou Session do Hibernate) fica associado à transação.
- Operações de escrita no banco são armazenadas em um cache transacional (primeiro nível do Hibernate) e só são enviadas ao banco no momento do commit.

---

**4. Commit ou Rollback**

Ao final da execução do método:

- **Commit**:
  - Se nenhuma exceção for lançada, o proxy confirma a transação:
    - As operações acumuladas são enviadas ao banco.
    - Os índices e constraints são validados pelo banco.
- **Rollback**:
  - Se uma exceção marcada como `@Transactional(rollbackFor)` ocorrer, o proxy realiza o rollback:
    - Todas as alterações são descartadas.
    - O estado do banco retorna ao momento do início da transação.

---

**5. Exceções e Rollback**

- Por padrão, o rollback ocorre somente para exceções **unchecked** (`RuntimeException` e `Error`).
- Para exceções **checked**, você deve especificá-las explicitamente usando `rollbackFor`:

  ```java
  @Transactional(rollbackFor = CustomException.class)
  public void myTransactionalMethod() { }
  ```

---

**6. Configurações Importantes**

- **Propagação**: Determina como uma transação interage com outra existente. Exemplos:
  - `REQUIRED`: Usa a transação existente ou cria uma nova.
  - `REQUIRES_NEW`: Sempre cria uma nova transação, suspendendo a atual.
- **Isolamento**: Define o nível de isolamento das transações:
  - `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`, etc.
- **Timeout**: Define o tempo máximo que uma transação pode durar antes de ser encerrada:

  ```java
  @Transactional(timeout = 30)
  ```

---

**7. Finalização**

Após o commit ou rollback, o contexto transacional é limpo do **ThreadLocal**, garantindo que futuras operações não sejam afetadas.

---

### Resumo

A anotação `@Transactional` automatiza o gerenciamento de transações, isolando o desenvolvedor de detalhes complexos, como início, commit e rollback. Internamente, o Spring usa proxies para interceptar métodos e gerenciar transações de forma transparente e eficiente, alinhando-se às configurações de propagação, isolamento e rollback definidas.

</details>

<details>
    <summary>
        <strong>
            5. Como você pode implantar um aplicativo Spring Boot econômico com recursos de servidor pagos conforme o uso?
        </strong>
    </summary>

</details>
