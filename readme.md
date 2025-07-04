# 🚀 @Transactional Mastery: Além do Básico

![Java](https://img.shields.io/badge/Java-17+-blue) ![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.x-brightgreen) ![License](https://img.shields.io/badge/License-MIT-green)

Este repositório contém o material de apoio e o código-fonte da apresentação **"@Transactional Mastery"**, um mergulho profundo no funcionamento da anotação `@Transactional` no Spring Framework.

A apresentação foi criada por **Bruno Scatena** ([github.com/bjscatena](https://github.com/bjscatena)).

---

## 📖 A Apresentação

O conteúdo completo da apresentação está no arquivo [**apresentacao.md**](./apresentacao.md).

Para a melhor experiência de visualização, com slides interativos, é recomendado usar a extensão **"Marp for VS Code"** no Visual Studio Code.

---

## 🎯 Tópicos Abordados

Esta apresentação cobre desde os conceitos fundamentais até os detalhes avançados e armadilhas comuns no uso de transações com Spring:

* **Fundamentos de Transações:** O que é uma transação e o problema clássico que ela resolve.
* **O Alicerce ACID:** Uma revisão dos quatro pilares:
    * Atomicidade
    * Consistência
    * Isolamento
    * Durabilidade
* **Níveis de Isolamento na Prática:**
    * Anomalias de Leitura: Dirty Read, Non-Repeatable Read e Phantom Read.
    * Configurando o `isolation` no `@Transactional`.
    * Particularidades do Oracle.
* **Propagation (Propagação):**
    * Entendendo como transações se comportam em chamadas de métodos aninhados.
    * Principais tipos: `REQUIRED`, `REQUIRES_NEW`, `SUPPORTS`, `NOT_SUPPORTED`, `MANDATORY` e `NEVER`.
* **Parâmetros Adicionais do `@Transactional`:**
    * `readOnly`: Para otimização de consultas.
    * `timeout`: Para controlar o tempo máximo de execução.
    * `rollbackFor`: Para customizar o comportamento de rollback com exceções.
* **O Padrão Spring Proxy:**
    * Como o Spring implementa o `@Transactional` por baixo dos panos.
    * Por que chamadas internas via `this` quebram o comportamento transacional.
    * Exemplos práticos de problemas e como evitá-los.

---

## 📄 Licença

Este projeto é distribuído sob a licença MIT. Veja o arquivo [LICENSE](./LICENSE) para mais detalhes.
