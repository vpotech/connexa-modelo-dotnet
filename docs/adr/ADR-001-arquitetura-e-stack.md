# ADR-001 — Monolito modular em .NET 10 com SPA em React

- **Situação:** aceita
- **Data:** 2026-11-09
- **Participantes:** Osi Paes Junior, Lucas Barros Simon, Guilherme Lins dos Santos, Guilherme Henrique Feitoza Correia, Rafael Alvarenga Kirsch Omelckzuk, João Del Manto Netto

## Contexto

O Connexa é uma plataforma de formação e acompanhamento de grupos de estudo, com territórios bem separados: perfil acadêmico, grupos, encontros, acompanhamento, painel e materiais. A equipe precisava escolher a arquitetura e a stack antes da Sprint 1.

Restrições consideradas:

- **Prazo e tamanho da equipe:** sprints de duas semanas, com seis integrantes que dedicam poucas horas por semana ao projeto.
- **Código escrito por agente de IA:** as Tasks são executadas por um agente (GitHub Copilot ou Codex). A arquitetura precisa impedir, de forma verificável, que o agente misture módulos ou espalhe regra de negócio.
- **Conhecimento prévio:** um integrante (Guilherme Lins dos Santos) já trabalha com C#, o que ajuda na revisão dos pull requests e no apoio aos demais.
- **Ambiente:** todos precisam rodar o projeto igual no GitHub Codespaces, na máquina local e no CI.

## Decisão

Adotamos a solução de arquitetura da disciplina na versão .NET, criando o repositório a partir do template `connexa-modelo-dotnet` sem adaptações: **monolito modular com fatias verticais** no back-end (.NET 10, C#, ASP.NET Core Minimal APIs, EF Core com PostgreSQL 17, um projeto por módulo) e uma **SPA em React 19** com TypeScript e Vite no front-end.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Template Java (`connexa-modelo-java`) | Mesma arquitetura; preferimos a stack em que a equipe já tem um integrante experiente (C#), o que ajuda na revisão dos pull requests do agente |
| Template Node.js (`connexa-modelo-node`) | Mesma arquitetura e mesma linguagem do front-end; pelo mesmo motivo, preferimos aproveitar o conhecimento de C# que já existe na equipe |
| Microsserviços | Exigiria vários deploys, comunicação pela rede e bancos separados, um custo operacional incompatível com o tamanho da equipe e com sprints de duas semanas |
| Monolito em camadas, sem módulos | Mais simples no início, mas não impede que o código de um território use os dados de outro; com o agente escrevendo o código, a mistura apareceria rápido e sem ninguém perceber |

## Consequências

- **Positivas:**
  - Cada módulo (`Connexa.<Modulo>`) é um projeto com tipos `internal` em `Internal/`, então o compilador impede um módulo de usar o interior de outro.
  - Um só deploy e um só banco, com um schema por módulo: simples de rodar no Codespaces e no CI.
  - O template já traz CI, testes de aceitação na API, verificação de fronteiras e as dependências da sprint, então a equipe começa direto pelas Tasks.
  - Experiência prévia de um integrante com C#, que pode apoiar as revisões.
- **Negativas:**
  - A maior parte da equipe precisa aprender C#, ASP.NET Core e EF Core durante as sprints.
  - O back-end (C#) e o front-end (TypeScript) usam linguagens diferentes.
  - Separar um módulo em um serviço próprio no futuro exigiria trocar eventos em processo por mensageria.
- **Como verificar:**
  - `ArquiteturaModularTest` (NetArchTest) falha se um módulo usar tipos de `Internal` de outro, se `Connexa.Comum` depender de um módulo de negócio ou se surgir dependência proibida.
  - O lint do front-end bloqueia importações de arquivos internos de outra feature.
  - As duas verificações rodam no `./verificar` e no CI de todo pull request, que a proteção do `main` exige antes do merge.
