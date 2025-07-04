---
marp: true
theme: default
class: lead, center, middle
---

<style>
section.shrink table {
  font-size: 0.6em;
}
</style>

![bg opacity:.5](https://github.com/user-attachments/assets/336c2570-5f05-422b-ba12-2a228d36ff9a)

# 🚀 @Transactional Mastery

### Além do básico: ACID, Isolamento, Propagation e Proxy

### **Bruno Scatena**  
🔗 [github.com/bjscatena](https://github.com/bjscatena)

---

# 📋 O que vamos ver

- Conceitos básicos e valores padrão do `@Transactional`  
- Problemas de concorrência e níveis de isolamento  
- Propagation: principais tipos e usos  
- Como o Spring usa proxy para transações  
- Por que chamadas internas via `this` falham  
- Exemplos práticos e soluções

---

# 🔄 O que é uma Transação?

- Uma **transação** é um conjunto de operações que são tratadas como uma única unidade lógica.
- Em resumo, uma transação garante que o sistema fique em um estado correto mesmo diante de falhas.

---

# 🏦 **Problema clássico**

1. Debitar R$100 da Conta A  
2. Creditar R$100 na Conta B

🔄 Operação deve ser **atômica e segura**.

---

# ⚡ **@Transactional no Spring**

```java
@Service
public class TransferenciaService {

    @Transactional
    public void executar(TransferenciaDTO dto) {
        // lógica para debitar e creditar
    }
}
```
---

# ⚙️ **@Transactional: Valores Padrão**
### Entendendo o comportamento padrão

Quando você usa `@Transactional` sem configurar nada, o Spring aplica:

- **Isolation:** `DEFAULT` (usa o nível padrão do banco, ex: READ_COMMITTED no Oracle/MySQL)  
- **Propagation:** `REQUIRED` (usa a transação existente ou cria uma nova)  
- **readOnly:** `false` (transação permite leitura e escrita)  
- **timeout:** indefinido (espera indefinidamente)  
- **rollbackFor:** só rola rollback para exceções unchecked (RuntimeException)

🔍 Saber isso ajuda a entender o que acontece “por trás dos panos” ao usar `@Transactional`.

---

# 🚀 **O Alicerce: ACID**

O contrato de garantias de toda transação

- **A**tomicidade
- **C**onsistência
- **I**solamento
- **D**urabilidade

---

# 🧩 **A**tomicidade
### Tudo ou Nada

Uma transação é indivisível.  
Ou todas as operações são executadas com sucesso, ou nenhuma é.  
**Não existe meio-sucesso.**

---

# 🛡️ **C**onsistência
### Mantendo as Regras

Assegura que o banco de dados vá de um estado **válido para outro válido**, respeitando todas as regras e restrições definidas.

---

# 🧭 **I**solamento
### Trabalhando em Silêncio

Garante que transações simultâneas **não interfiram** entre si.  
As alterações de uma transação **ficam invisíveis** para as outras até serem confirmadas.

---

# 🗿 **D**urabilidade
### Escrito em Pedra

Após o `COMMIT`, as mudanças se tornam **permanentes** e resistem a falhas, como quedas de energia ou travamentos.

---

# 🔍 **O "I" do ACID na Prática**
### Níveis de Isolamento

Se o **Isolamento** existe, por que ainda temos problemas de concorrência?

✅ **Isolamento** vs 🐢 **Performance**

<br>🦸 **Níveis de Isolamento** ajudam a equilibrar essa balança.

---

# 💩 Dirty Read

Quando uma transação lê dados alterados por outra transação que **ainda não foram confirmados (commit)**.  
Se a outra transação fizer rollback, a leitura foi de um dado que **nunca existiu**.

---

<!--
class: shrink
-->
# 💩 Dirty Read

| **Transação A** | **Transação B** | **Observação** |
|-----------------|-----------------|----------------|
| `BEGIN` | | T1 inicia |
| `UPDATE conta SET saldo = saldo - 100 WHERE id = 1` | | T1 debita 100, mas **não commita** |
| | `BEGIN` | T2 inicia |
| | `SELECT saldo FROM conta WHERE id = 1` | T2 lê o saldo já alterado por T1 (**dirty read**) |
| `ROLLBACK` | | T1 faz rollback, saldo volta ao original |
| | | T2 leu um valor que **nunca existiu** no banco |

---

# 🔁 Non-Repeatable Read

Ocorre quando uma transação lê um dado, outra transação o altera e confirma, e ao reler, o valor mudou.  
A mesma consulta retorna **resultados diferentes** na mesma transação.

---
<!--
class: shrink
-->

# 🔁 Non-Repeatable Read

| **Transação A** | **Transação B** | **Observação** |
|-----------------|-----------------|----------------|
| `BEGIN` | | T1 inicia |
| `SELECT saldo FROM conta WHERE id = 1` | | T1 lê saldo = 500 |
| | `BEGIN` | T2 inicia |
| | `UPDATE conta SET saldo = 600 WHERE id = 1` | T2 altera saldo |
| | `COMMIT` | T2 confirma alteração |
| `SELECT saldo FROM conta WHERE id = 1` | | T1 relê e vê saldo = 600 (**valor mudou**) |
| `COMMIT` | | T1 finaliza |

---

# 👻 Phantom Read

Acontece quando uma transação lê um conjunto de linhas com um filtro, outra transação insere (ou deleta) linhas que também satisfazem esse filtro, e ao reler, a primeira transação vê **linhas novas ou faltantes**.

---
<!--
class: shrink
-->
# 👻 Phantom Read

| **Transação A** | **Transação B** | **Observação** |
|-----------------|-----------------|----------------|
| `BEGIN` | | T1 inicia |
| `SELECT * FROM pedidos WHERE valor > 100` | | T1 lê 3 pedidos |
| | `BEGIN` | T2 inicia |
| | `INSERT INTO pedidos (id, valor) VALUES (999, 150)` | T2 insere novo pedido |
| | `COMMIT` | T2 confirma |
| `SELECT * FROM pedidos WHERE valor > 100` | | T1 relê e vê 4 pedidos (**linha fantasma apareceu**) |
| `COMMIT` | | T1 finaliza |

---

# 🛡️ **@Transactional: Isolation**
### Controle de concorrência

O `@Transactional` permite configurar o **nível de isolamento** da transação:

- `READ_UNCOMMITTED`
- `READ_COMMITTED`
- `REPEATABLE_READ`
- `SERIALIZABLE`

🔍 Cada nível controla **quais problemas de leitura** podem ocorrer em transações simultâneas.

---

# 🧩 **Isolation Levels x Problemas**

| **Isolation**           | **Previne**               |
|--------------------------|----------------------------|
| `READ_UNCOMMITTED` | Nada (permite dirty read) |
| `READ_COMMITTED`   | Dirty read                 |
| `REPEATABLE_READ`  | Dirty read, non-repeatable read |
| `SERIALIZABLE`     | Dirty read, non-repeatable read, phantom read |

💡 **Quanto maior o isolamento, menor a concorrência e maior a segurança.**

---

# 🏛️ **Oracle e Isolation Levels**

- O Oracle **não suporta** `READ_UNCOMMITTED`.  
- O nível mínimo é **`READ_COMMITTED`** (padrão).  
- **Não existe `REPEATABLE_READ`** no Oracle; ele vai direto para **`SERIALIZABLE`**.

⚠️ Isso deve ser considerado ao configurar isolamento no Spring com Oracle.

---

# 🔄 **Propagation**
### O que é e por que importa?

Propagation controla como uma transação se comporta quando um método transacional chama outro método transacional.

➡️ Essencial para definir se a transação atual será usada, criada ou suspensa.

---

# 📋 **Principais tipos de Propagation**

- `REQUIRED` (padrão)  
- `REQUIRES_NEW`  
- `SUPPORTS`  
- `NOT_SUPPORTED`  
- `MANDATORY`  
- `NEVER`

---

# ✅ Propagation: REQUIRED
### Usa a transação existente ou cria uma nova

- Se já existe uma transação ativa, o método participa dela.  
- Se não, cria uma nova transação.  
- É o comportamento padrão do `@Transactional`.

🔹 Ideal para garantir atomicidade em operações comuns.

---

# 🆕 Propagation: REQUIRES_NEW
### Cria uma nova transação independente

- Suspende a transação atual (se existir).  
- Executa o método em uma transação nova e separada.  
- Útil quando a operação precisa ser commitada imediatamente, mesmo que a transação original falhe.

🔹 Exemplo: registrar logs ou auditorias independentemente da transação principal.

---

# 💡 Exemplo real: Quando usar REQUIRES_NEW
### Processamento de pagamentos + registro de auditoria

- Imagine um sistema que processa pagamentos em uma transação principal.  
- Em paralelo, precisa registrar uma auditoria detalhada em banco, que não pode ser perdida.  
- Usamos `REQUIRES_NEW` para o método de auditoria:

  - A auditoria roda em uma nova transação independente.  
  - Mesmo que o pagamento principal falhe e faça rollback, o registro de auditoria **é salvo**.  
  - Garante rastreabilidade e compliance, mesmo em falhas.

🔹 Evita perder logs importantes por falhas na transação principal.

---

# 🤝 Propagation: SUPPORTS
### Participa se houver transação, senão roda sem

- Se uma transação já está ativa, o método participa dela.  
- Se não houver transação, o método executa sem transação.  
- Útil para métodos que podem ser chamados tanto dentro quanto fora de contexto transacional.

🔹 Flexível, evita erros e permite reutilização em diferentes cenários.

---

# ⏸️ Propagation: NOT_SUPPORTED
### Sempre executa fora de uma transação

- Se uma transação estiver ativa, ela será suspensa durante a execução do método.  
- O método roda sem nenhuma transação.  
- Ideal para operações que não devem ser afetadas por transações, como chamadas externas ou tarefas que não precisam de atomicidade.

🔹 Evita impactos da transação em operações específicas.

---

# 📧 Exemplo real: Uso de NOT_SUPPORTED
### Envio de e-mail fora da transação principal

- Após uma transferência bancária concluída, o sistema envia um e-mail de confirmação.  
- O método de envio é anotado com `@Transactional(propagation = NOT_SUPPORTED)`.  
- Assim, o envio de e-mail **não roda dentro da transação do banco**.  
- Se o envio falhar ou atrasar, a transação principal **não é afetada nem travada**.

🔹 Evita que problemas externos prejudiquem operações críticas.

---

# 🚫 Propagation: NEVER
### Nunca rode dentro de uma transação

- Garante que o método **não execute dentro de uma transação ativa**.  
- Se houver uma transação em andamento, o Spring lança uma exceção.  
- Ideal para métodos que fazem chamadas externas lentas, como APIs remotas, evitando travar o pool de conexões.  
- Ajuda a evitar bloqueios e lentidão causada por operações externas em transações.

🔹 Use `NEVER` para garantir isolamento dessas operações.

---

# 📖 @Transactional: readOnly
### Otimizando transações para consultas

- `readOnly = true` indica que a transação será usada **apenas para leitura**.  
- O banco pode otimizar a execução, por exemplo:  
  - Desabilitando locks desnecessários  
  - Melhorando desempenho  
- Ajuda a evitar alterações acidentais nos dados durante a transação.  
- Por padrão, `readOnly = false` (permite escrita).

🔹 Use em métodos que só consultam dados.

---

# ⏱️ @Transactional: timeout
### Controlando o tempo máximo da transação

- Define o tempo máximo (em segundos) que a transação pode ficar ativa.  
- Se o tempo for excedido, o Spring **faz rollback automático** da transação.  
- Evita travamentos e bloqueios prolongados no banco.  
- Útil para operações longas que podem travar recursos.

🔹 Exemplo: `@Transactional(timeout = 5)` — transação expira após 5 segundos.

---

# 🔄 @Transactional: rollbackFor
### Controlando quais exceções disparam rollback

- Por padrão, o Spring faz rollback apenas para **exceções do tipo RuntimeException** (unchecked).  
- Com `rollbackFor`, você pode especificar **quais exceções checked também devem causar rollback**.  
- Útil para garantir consistência quando métodos lançam exceções verificadas.

🔹 Exemplo: `@Transactional(rollbackFor = IOException.class)`

---

# 🛡️ O Padrão Proxy no Spring
### Como o Spring implementa o `@Transactional`

- O Spring utiliza o **padrão de projeto Proxy** para aplicar funcionalidades transversais como transações, cache e segurança.
- Ao anotar um método com `@Transactional`, o Spring **não modifica a classe original**. Em vez disso, ele cria um **proxy** que envolve a classe original.
- Esse proxy intercepta as chamadas aos métodos anotados, permitindo que o Spring gerencie a transação antes e depois da execução do método.

🔹 Compreender o funcionamento do proxy é essencial para entender como o Spring gerencia transações e outras funcionalidades transversais.

---

# 🕰️ Antes do Spring: Gerenciamento Manual com JDBC
### Exemplo básico de transação manual

```java
Connection conn = dataSource.getConnection();
try {
    conn.setAutoCommit(false);

    // Debitar valor
    debitarConta(conn, contaOrigem, valor);

    // Creditar valor
    creditarConta(conn, contaDestino, valor);

    conn.commit();
} catch (SQLException e) {
    conn.rollback();
    throw e;
} finally {
    conn.close();
}
```

---

# 💸 Exemplo básico: Débito e Crédito sem proxy
### Implementação manual sem controle automático de transação

```java
public class ContaService {

    private void debitar(Conta conta, BigDecimal valor) {
        conta.setSaldo(conta.getSaldo().subtract(valor));
    }

    private void creditar(Conta conta, BigDecimal valor) {
        conta.setSaldo(conta.getSaldo().add(valor));
    }

    public void transferir(Conta origem, Conta destino, BigDecimal valor) {
        debitar(origem, valor);
        creditar(destino, valor);
    }
}
```

---

# 🛡️ Proxy Manual: Simulando @Transactional
### Proxy que gerencia transação envolvendo `transferir`

```java

@Primary
@Service
public class ContaServiceProxy extends ContaService {

    @Override
    public void transferir(Conta origem, Conta destino, BigDecimal valor) {
        System.out.println("Iniciando transação manual...");
        try {
            super.transferir(origem, destino, valor);
            System.out.println("Commit da transação");
        } catch (Exception e) {
            System.out.println("Rollback da transação");
            throw e;
        }
    }
}
```
---
# 🛡️ Proxy Manual Real: Gerenciando Transação com TransactionManager
### Simulação próxima ao comportamento do Spring

```java

@Primary
@Service
public class ContaServiceProxy extends ContaService {

    private PlatformTransactionManager transactionManager;

    public ContaServiceProxy(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    @Override
    public void transferir(Conta origem, Conta destino, BigDecimal valor) {
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        def.setName("transferirTransaction");
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

        TransactionStatus status = transactionManager.getTransaction(def);

        try {
            super.transferir(origem, destino, valor);
            transactionManager.commit(status);
        } catch (Exception e) {
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```
---

# ⚙️ Configuração de Bean: Spring injeta o Proxy
### Exemplo simples usando @Primary

```java
@Configuration
public class AppConfig {

    @Bean
    public ContaService contaService(PlatformTransactionManager txManager) {        
        return new ContaServiceProxy(txManager);
    }
}
```
---

# ⚠️ Por que chamadas via `this` não funcionam?
- O Spring usa um proxy para aplicar `@Transactional`.
- Chamadas externas passam pelo proxy e funcionam.
- Chamadas internas via `this` **não passam pelo proxy**.
- Resultado: anotações como `@Transactional` não são aplicadas.
  
✅ Para evitar isso:  
- Use self-injection (`@Autowired private SeuService self;`) e chame `self.metodo()`.  
- Ou mova o método para outro bean.

---


# ⚠️ Rollback falha ao chamar método transacional via `this`

```java
@Service
public class PedidoService {

    public void processarPedido(Pedido pedido) {
        // método público não transacional

        // chama método transacional via this — NÃO passa pelo proxy
        this.salvarPedido(pedido);
    }

    @Transactional
    public void salvarPedido(Pedido pedido) {
        // Salva o pedido no banco
        pedidoRepository.save(pedido);

        // Atualiza o estoque
        estoqueService.diminuirEstoque(pedido.getProdutoId(), pedido.getQuantidade());

        // Validação crítica
        if (pedido.getQuantidade() <= 0) {
            throw new RuntimeException("Quantidade inválida!");
        }
    }
}
```

---

# ⚠️ `Propagation.NOT_SUPPORTED` ignorado por chamada via this
### Método para chamada externa lenta que deveria suspender transação

```java
@Service
public class PagamentoService {

    @Transactional
    public void processarPagamento(Pagamento pagamento) {
        debitarConta(pagamento.getContaOrigem(), pagamento.getValor());

        // Chamada via this ignora o comportamento NOT_SUPPORTED
        this.enviarSmsConfirmacao(pagamento.getTelefoneCliente(), pagamento.getValor());
    }

    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void enviarSmsConfirmacao(String telefone, BigDecimal valor) {
        // Chamada externa lenta para serviço de SMS
        smsGateway.enviarSms(telefone, "Pagamento de R$" + valor + " realizado com sucesso.");
        System.out.println("SMS de confirmação enviado para " + telefone);
    }

    private void debitarConta(Conta conta, BigDecimal valor) {
        // lógica de débito na conta
    }
}
```
---

# 🚀 **Resumo & Próximos Passos**

- `@Transactional` é essencial para garantir **atomicidade** e **consistência**  
- Entender **isolamento** e **propagation** evita problemas comuns em concorrência  
- O Spring usa **proxy** para aplicar transações, atenção às chamadas internas via `this`  
- Sempre valide a arquitetura para garantir que o `@Transactional` funcione como esperado  

---

### Próximos passos:

✅ Revisite seus serviços para evitar chamadas internas diretas  
✅ Use self-injection ou separe métodos transacionais em beans distintos  
✅ Configure isolamento e propagations conforme a necessidade do negócio  
✅ Teste cenários de concorrência para evitar surpresas em produção

---

# 🎯 Obrigado!

**Bruno Scatena**

[https://github.com/bjscatena](https://github.com/bjscatena)


