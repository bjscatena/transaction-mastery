---
marp: true
theme: default
class: lead, center, middle
---

<style>
section.shrink table {
  font-size: 0.6em;
}

section.small {
  font-size: 0.8em;
}  
</style>

![bg right](https://github.com/user-attachments/assets/d8a14318-8ca7-4fe1-b364-2e05db97636f)


# 🚀 @Transactional Mastery

### Além do básico

### **Bruno Scatena**  
🔗 [github.com/bjscatena](https://github.com/bjscatena)

---

# 📋 O que vamos ver

- ACID
- Isolamento  
- Propagation
- Spring Proxy
- Usos incorretos e problemas

---

# 🔄 O que é uma Transação?

- Conjunto de operações que são tratadas como uma única unidade lógica.
- Garante que o sistema fique em um estado correto mesmo diante de falhas.

---

# 🏦 Problema clássico

1. Debitar R$100 da Conta A  
2. Creditar R$100 na Conta B

🔄 Operação deve ser **atômica e segura**.

---

# 🚀 O Alicerce: ACID

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

# 🔍 O "I" do ACID na Prática
### Níveis de Isolamento

Se o **Isolamento** existe, por que ainda temos problemas de concorrência?

✅ **Isolamento** vs 🐢 **Performance**

<br>🦸 **Níveis de Isolamento** ajudam a equilibrar essa balança.

![bg left:40%](https://github.com/user-attachments/assets/fe4e61e4-2045-44ef-a169-e7368dfa716e)

---

# ⚠️ Anomalias em Transações

Quando múltiplas transações rodam simultaneamente, podem surgir problemas de **concorrência** que afetam a **consistência** dos dados.

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

# 🛡️ @Transactional: Isolation
### Controle de concorrência

O `@Transactional` permite configurar o **nível de isolamento** da transação:

- `READ_UNCOMMITTED`
- `READ_COMMITTED`
- `REPEATABLE_READ`
- `SERIALIZABLE`

🔍 Cada nível controla **quais problemas de leitura** podem ocorrer em transações simultâneas.

---

# 🧩 Isolation Levels x Problemas

| **Isolation**           | **Previne**               |
|--------------------------|----------------------------|
| `READ_UNCOMMITTED` | Nada (permite dirty read) |
| `READ_COMMITTED`   | Dirty read                 |
| `REPEATABLE_READ`  | Dirty read, non-repeatable read |
| `SERIALIZABLE`     | Dirty read, non-repeatable read, phantom read |

💡 **Quanto maior o isolamento, menor a concorrência e maior a segurança.**

---

# 🏛️ Oracle e Isolation Levels

- O Oracle **não suporta** `READ_UNCOMMITTED`.  
- O nível mínimo é **`READ_COMMITTED`** (padrão).  
- **Não existe `REPEATABLE_READ`** no Oracle; ele vai direto para **`SERIALIZABLE`**.

⚠️ Isso deve ser considerado ao configurar isolamento no Spring com Oracle.

---

# 🔄 Propagation
### O que é e por que importa?

Propagation controla como uma transação se comporta quando um método transacional chama outro método transacional.

➡️ Essencial para definir se a transação atual será usada, criada ou suspensa.

---

# 📋 Principais tipos de Propagation

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

# 🤝 Propagation: SUPPORTS
### Participa se houver transação, senão roda sem

- Se uma transação já está ativa, o método participa dela.  
- Se não houver transação, o método executa sem transação.  
- Útil para métodos que podem ser chamados tanto dentro quanto fora de contexto transacional.

🔹 Flexível, evita erros e permite reutilização em diferentes cenários.

---

# ⚠️ Propagation: MANDATORY
### Exige que uma transação já esteja ativa

- O método **deve ser chamado dentro de uma transação existente**.  
- Se não houver transação ativa, o Spring lança uma **IllegalTransactionStateException**.  
- Útil para garantir que certas operações **não sejam executadas fora do contexto transacional**.  
- Ajuda a impor regras rígidas sobre o fluxo transacional.

### Quando usar?

- Métodos que fazem parte obrigatória de uma transação maior.  
- Serviços que não podem operar isoladamente.

---

# ⏸️ Propagation: NOT_SUPPORTED
### Sempre executa fora de uma transação

- Se uma transação estiver ativa, ela será suspensa durante a execução do método.  
- O método roda sem nenhuma transação.  
- Ideal para operações que não devem ser afetadas por transações, como chamadas externas ou tarefas que não precisam de atomicidade.

🔹 Evita impactos da transação em operações específicas.

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

# ⚙️ @Transactional: Valores Padrão
### Entendendo o comportamento padrão

Quando você usa `@Transactional` sem configurar nada, o Spring aplica:

- **Isolation:** `DEFAULT` (usa o nível padrão do banco, ex: READ_COMMITTED no Oracle/MySQL)  
- **Propagation:** `REQUIRED` (usa a transação existente ou cria uma nova)  
- **readOnly:** `false` (transação permite leitura e escrita)  
- **timeout:** indefinido (espera indefinidamente)  
- **rollbackFor:** só rola rollback para exceções unchecked (RuntimeException)

🔍 Saber isso ajuda a entender o que acontece “por trás dos panos” ao usar `@Transactional`.

---

# 🚀 Casos Reais de Uso do `@Transactional`

<style>
ul.custom-list {
  list-style: none;
  padding-left: 0;
}
ul.custom-list li {
  margin-bottom: 1.2em;
  font-size: 1.1em;
}
.code {
  background: #272822;
  color: #f8f8f2;
  padding: 0.2em 0.5em;
  border-radius: 4px;
  font-family: 'Courier New', Courier, monospace;
  font-weight: bold;
}
.highlight {
  color: #f39c12;
  font-weight: bold;
}
</style>

<ul class="custom-list">

<li>🏦 <span class="highlight">Transferência bancária</span><br>
<span class="code">@Transactional(propagation = REQUIRED)</span><br>
Garantia de atomicidade — rollback se falhar.</li>

<li>📧 <span class="highlight">Envio de e-mail</span><br>
<span class="code">@Transactional(propagation = NOT_SUPPORTED)</span><br>
Evita travar a transação principal por operações externas lentas.</li>

<li>📝 <span class="highlight">Registro de auditoria</span><br>
<span class="code">@Transactional(propagation = REQUIRES_NEW)</span><br>
Persistência independente, mesmo se a transação principal falhar.</li>

<li>📊 <span class="highlight">Consulta de relatórios</span><br>
<span class="code">@Transactional(readOnly = true)</span><br>
Otimização para consultas, evitando locks desnecessários.</li>

<li>⏳ <span class="highlight">Processamento batch lento</span><br>
<span class="code">@Transactional(timeout = 300)</span><br>
Limita tempo máximo para evitar travamentos no sistema.</li>

</ul>

---

# 🛡️ O Padrão Proxy no Spring
### Como o Spring implementa o `@Transactional`

- O Spring utiliza o **padrão de projeto Proxy** para aplicar funcionalidades transversais como transações, cache e segurança.
- Ao anotar um método com `@Transactional`, o Spring **não modifica a classe original**. Em vez disso, ele cria um **proxy** que envolve a classe original.
- Esse proxy intercepta as chamadas aos métodos anotados, permitindo que o Spring gerencie a transação antes e depois da execução do método.

🔹 Compreender o funcionamento do proxy é essencial para entender como o Spring gerencia transações e outras funcionalidades transversais.

---
<!--
class: small
-->

# 🕰️ Antes do Spring

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

<!--
class: small
-->
# 💸 Exemplo básico: Débito e Crédito sem proxy

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
<!--
class: small
-->
# 🛡️ Proxy Manual

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
<!--
class: small
-->
# ⚙️ Configuração de Bean

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
<!--
class: small
-->

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
<!--
class: small
-->
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


