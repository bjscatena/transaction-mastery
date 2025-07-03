---
marp: true
theme: uncover
class:
  - lead
paginate: true
---
<style>
section {
  background: #282a36;
  color: #f8f8f2;
  padding: 60px;
}

h1, h2, h3 {
  color: #bd93f9; /* Roxo */
  text-align: left;
  border-bottom: none;
  background: none;
}

a {
  color: #8be9fd; /* Ciano */
}

strong {
    color: #50fa7b; /* Verde */
}

code {
    background-color: #44475a;
    border-radius: 5px;
    padding: 2px 5px;
}
</style>

# @Transactional mastery
## Além do padrão

**Seu Nome**
Engenheiro de Software Sênior

---

![bg right:40% fit](https://i.imgur.com/g1f52Yq.png)

# O Problema do Dia a Dia

### E se o PIX falhasse no meio?

O dinheiro sai da sua conta...
...mas nunca chega ao destino.

O que impede esse caos de acontecer milhões de vezes por dia é o conceito de **Transação**.

---

# O `@Transactional` que Vivemos

A gente resolve isso com uma palavra mágica: `@Transactional`.

```java
@Service
public class PagamentoService {

    @Transactional
    public void executarPagamento(PagamentoDTO dto) {
        // ... lógica para debitar e creditar ...
    }
}
