---
marp: true
theme: uncover
class:
  - lead
footer: 'Seu Nome | Seu GitHub: github.com/seu-usuario'
paginate: true
---

# **Desvendando o `@Transactional`**
## Muito além do padrão

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
