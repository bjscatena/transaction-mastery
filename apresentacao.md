---
marp: true
theme: uncover
---

# @Transactional mastery
## Além do padrão

**Bruno Scatena**

*https://github.com/bjscatena*

---

# O Problema Universal

### A transferência bancária

Imagine o fluxo de uma transferência:
1. Debitar R$100 da Conta A.
2. Creditar R$100 na Conta B.

E se o sistema cair exatamente entre o passo 1 e o 2?

O dinheiro sumiu da conta A, mas nunca chegou na B. Como garantimos que operações críticas como essa sejam seguras? **Transação**.

---

# O `@Transactional` Padrão

No nosso dia a dia, usamos a anotação `@Transactional` e confiamos que a mágica aconteça.

```java
@Service
public class TransferenciaService {

    @Transactional
    public void executar(TransferenciaDTO dto) {
        // ... lógica para debitar e creditar ...
    }
}
