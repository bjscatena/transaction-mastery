---
marp: true
theme: uncover
class:
  - lead
paginate: true
---
<style>
section {
  padding: 60px;
}

</style>

# @Transactional mastery
## Além do padrão

**Bruno Scatena**

---

---

# **O Alicerce de Tudo: ACID**

Toda a "mágica" de uma transação segura se apoia em 4 pilares,
um acrônimo que garante a paz de espírito de todo desenvolvedor.

- **A**tomicidade
- **C**onsistência
- **I**solamento
- **D**urabilidade

---

![bg right:30% fit](https://i.imgur.com/pL8d9t6.png)

# **A**tomicidade
### Tudo ou Nada

A transação é uma unidade indivisível. Ou todas as operações dentro dela funcionam, ou nenhuma funciona.

**Analogia (Mágica):**
Pense num truque de mágica. Ou o coelho sai da cartola com sucesso (**COMMIT**), ou o truque dá errado e tudo volta exatamente como era antes, sem nenhum vestígio (**ROLLBACK**).

Não existe "meio coelho" fora da cartola.

---

![bg right:30% fit](https://i.imgur.com/2qK3jA2.png)

# **C**onsistência
### As Regras do Jogo

A transação sempre levará o banco de dados de um estado válido para outro, respeitando todas as regras de negócio definidas.

**Analogia (Kickboxing):**
A transação é o juiz da luta. Ela garante que nenhuma regra seja violada (ex: um saldo ficar negativo). Se uma regra for quebrada, a transação inteira é invalidada, como um golpe ilegal que anula o round.

---

![bg right:30% fit](https://i.imgur.com/j3yY2rJ.png)

# **I**solamento
### Cada Um no Seu Canto

Transações que rodam ao mesmo tempo não podem interferir umas nas outras. Cada uma opera na sua própria "bolha", como se estivesse sozinha no sistema.

**Analogia (Videogame):**
Imagine dois amigos jogando o mesmo jogo, cada um no seu PS5. O progresso e a bagunça que um faz no seu "save" (sua transação) não afetam em nada o jogo do outro.

---

![bg right:30% fit](https://i.imgur.com/lJ4jO3h.png)

# **D**urabilidade
### Salvou, tá Salvo!

Uma vez que a transação é confirmada com um **COMMIT**, a mudança é permanente e sobreviverá a qualquer falha, seja uma queda de energia ou o sistema travar.

**Analogia (Videogame):**
É o **save point**. Depois que o ícone de "salvando" some da tela, seu progresso está gravado na memória para sempre. Pode desligar o console, a luz pode acabar, mas quando você voltar, seu progresso estará lá.

---
