---
marp: true
theme: default
---

<style>
section.shrink table {
  font-size: 0.6em;
}
</style>

# @Transactional mastery
## Além do padrão

**Bruno Scatena**

*https://github.com/bjscatena*

---

# Problema básico

1. Debitar R$100 da Conta A.
2. Creditar R$100 na Conta B.

---

# `@Transactional`

```java
@Service
public class TransferenciaService {

    @Transactional
    public void executar(TransferenciaDTO dto) {
        // ... lógica para debitar e creditar ...
    }
}
```

---

# **O Alicerce: ACID**

Contrato de garantias

- **A**tomicidade
- **C**onsistência
- **I**solamento
- **D**urabilidade

![bg right](https://github.com/user-attachments/assets/8ee6f3d2-6fb3-45ac-9ea0-84322b67341c)

---

# **A**tomicidade
### Tudo ou Nada

Uma transação é uma operação atômica. Ou todas as suas partes são executadas com sucesso, ou nenhuma delas é. Não existe "meio-sucesso".

---

# **C**onsistência
### Mantendo as Regras

A transação garante que o banco de dados sempre saia de um estado válido para outro estado válido, respeitando todas as regras definidas.

---

# **I**solamento
### Trabalhando em Silêncio

Garante que transações rodando ao mesmo tempo não interfiram umas nas outras. O trabalho de uma transação fica invisível para as outras até que seja concluído.

---

# **D**urabilidade
### Escrito em Pedra

Uma vez que a transação é confirmada (`COMMIT`), a mudança é permanente e sobreviverá a qualquer falha do sistema, como uma queda de energia.

---

---

# **O "I" do ACID na Prática**
### Níveis de Isolamento

Se o "I" garante Isolamento, por que ainda existem problemas de concorrência?

:white_check_mark: Isolamento X 🐢 Performance

<br>🦸 **Níveis de Isolamento**.

![bg left](https://github.com/user-attachments/assets/b29e78ae-dc57-426d-93c9-f48e17978ba1)


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
