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

Porque o isolamento perfeito custa caro em performance. Para equilibrar, os bancos de dados nos oferecem **Níveis de Isolamento**.

![bg left](https://github.com/user-attachments/assets/b29e78ae-dc57-426d-93c9-f48e17978ba1)


---

# Dirty reads

Dirty read ocorre quando uma transação lê dados alterados por outra transação que ainda não foram confirmados. Se a primeira transação for revertida, a segunda terá lido dados inválidos.

---
<div>

### Transação A
#### (Relatório do Site)

`ISOLATION = READ_UNCOMMITTED`

**3.** Inicia a leitura para mostrar os preços na vitrine do site.

**4.** `SELECT preco FROM produto WHERE id = 123;`
   - **Resultado: `R$ 10,00`**

**5.** O site exibe o produto por **R$ 10,00** para todos os clientes!

</div>

<div>

### Transação B
#### (Promoção Relâmpago)

**1.** **INICIA TRANSAÇÃO**

**2.** Aplica um desconto agressivo.
   `UPDATE produto SET preco = 10.00 WHERE id = 123;`
   *(Ainda não commitou!)*

**6.** Ocorre um erro de negócio! A promoção é inviável.
   **`ROLLBACK;`**

**7.** O preço do produto volta a ser o original no banco.

</div>

---

# O Desastre do Dirty Read

### O que aconteceu?

A **Transação A** leu um dado "sujo", um valor que existiu por um instante mas que nunca foi confirmado.

**Resultado:** O site anunciou um produto por um preço extremamente baixo que, na realidade, nunca foi válido. Isso gera frustração no cliente e um problema de negócio.

É por isso que o nível `READ_UNCOMMITTED` é muito perigoso e quase nunca utilizado.

---

