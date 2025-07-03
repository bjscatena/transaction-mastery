---
marp: true
theme: default
---

<style>
section.shrink {
  font-size: 0.75em; /* <<-- AJUSTE ESTE VALOR SE PRECISAR */
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

# Dirty reads

Dirty read ocorre quando uma transação lê dados alterados por outra transação que ainda não foram confirmados. Se a primeira transação for revertida, a segunda terá lido dados inválidos.

---

<!--
class: shrink
-->

# Anomalia: Leitura Suja (Dirty Read)

| Passo | Transação A (Site)<br>`READ_UNCOMMITTED` | Transação B (Promo) | Banco<br>(Preço Real) |
| :--- | :--- | :--- | :--- |
| **1** | | `BEGIN;` | `R$ 100,00` |
| **2** | | `UPDATE preco = 10.00;` | `R$ 100,00` (dado "sujo") |
| **3** | `BEGIN;` | | `R$ 100,00` |
| **4** | `SELECT preco;` <br> ➡️ **Lê `R$ 10,00`** | | `R$ 100,00` |
| **5** | **Exibe R$10,00 no site!** | | `R$ 100,00` |
| **6** | | ⚠️ **Erro!** <br> `ROLLBACK;` | `R$ 100,00` |
| **7** | `COMMIT;` | | `R$ 100,00` |
| **Resumo:** | **Anunciou preço falso!** | **Operação desfeita.** | **Preço nunca foi R$10.** |

---

# Non-Repeatable Reads

Non-repeatable read ocorre quando uma transação lê um dado, outra transação o altera e, ao reler, o valor mudou. Diferente de dirty read, aqui o dado já foi commitado, mas causa problemas por mudar entre leituras.

---

# Anomalia: Leitura Não Repetível

| Passo | Transação A (Lógica de Venda - `READ_COMMITTED`) | Transação B (Ajuste de Estoque) | Estado **Real** no Banco (Estoque) |
| :--- | :--- | :--- | :--- |
| **1** | `BEGIN;` | | `10` |
| **2** | `SELECT estoque FROM produto WHERE id=123;` <br> ➡️ **Lê `10`** | | `10` |
| **3** | *(Faz uma verificação de regra de negócio...)* | `BEGIN;` | `10` |
| **4** | | `UPDATE produto SET estoque = 9 WHERE id=123;` | `10` (Mudança da B não commitada) |
| **5** | | `COMMIT;` | **`9`** |
| **6** | *(Precisa verificar o estoque de novo...)*<br>`SELECT estoque FROM produto WHERE id=123;` <br> ➡️ **Lê `9`** | | `9`|
| **7** | `// O valor mudou no meio da minha transação!` | | `9`|
| **Resultado:** | **Lógica inconsistente!** | **Operação bem-sucedida.** | **Dado foi alterado.**|

<br>

---
