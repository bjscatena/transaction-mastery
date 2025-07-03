---
marp: true
theme: default
---

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

**Analogia (Transferência Bancária):**
É a própria transferência. Ou o débito E o crédito funcionam (**COMMIT**), ou, se qualquer um falhar, a operação inteira é desfeita e o saldo original é restaurado (**ROLLBACK**).

---

# **C**onsistência
### Mantendo as Regras

A transação garante que o banco de dados sempre saia de um estado válido para outro estado válido, respeitando todas as regras definidas.

**Analogia (Sudoku):**
Cada número que você adiciona deve seguir as regras. Uma transação só pode ser concluída se o "tabuleiro" do banco de dados continuar válido ao final. Uma tentativa de duplicar uma chave primária, por exemplo, quebraria a consistência.

---

# **I**solamento
### Trabalhando em Silêncio

Garante que transações rodando ao mesmo tempo não interfiram umas nas outras. O trabalho de uma transação fica invisível para as outras até que seja concluído.

**Analogia (Prova em Sala de Aula):**
Vários alunos (transações) estão fazendo uma prova, cada um na sua folha. Você não vê as respostas do seu colega (os dados "sujos" dele) enquanto ele ainda está escrevendo. Você só vê o resultado final quando ele entrega a prova (`COMMIT`).

---

# **D**urabilidade
### Escrito em Pedra

Uma vez que a transação é confirmada (`COMMIT`), a mudança é permanente e sobreviverá a qualquer falha do sistema, como uma queda de energia.

**Analogia (Assinatura de Contrato):**
Depois que um contrato legal é assinado, ele se torna um registro oficial e durável. As cláusulas ali contidas são permanentes. O `COMMIT` é a assinatura final que torna os dados permanentes no banco.

---
