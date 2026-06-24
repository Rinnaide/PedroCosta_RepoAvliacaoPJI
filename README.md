# 🏫 Caça aos Erros — Sistema Escolar de Cadastro
**Instituição:** Senac  
**Avaliação:** Projeto Integrador  
**Aluno:** Pedro Costa  
---
# 📋 README — Modificações do Projeto
> Registro das correções aplicadas ao projeto, com descrição objetiva do que foi alterado e o motivo técnico de cada mudança.
---
## 📂 Arquivos Modificados
| # | Arquivo | Tipo de Correção |
|---|---------|-----------------|
| 1 | `controller/AlunoController.java` | Verbo HTTP e tipo de parâmetro |
| 2 | `controller/ProfessorController.java` | Transação ausente |
| 3 | `controller/MatriculaController.java` | Tipo de ID incompatível |
| 4 | `model/aluno/Aluno.java` | Mapeamento de tabela errado |
| 5 | `model/aluno/DadosListagemAluno.java` | Campos invertidos no DTO |
| 6 | `model/professor/Professor.java` | Campo errado no update |
---
## 🔧 Detalhamento das Alterações
### 1 · `AlunoController.java`
**Caminho:** `src/main/java/com/github/app/controller/AlunoController.java`
#### Alterações
```diff
- @PostMapping
+ @PutMapping
  public ResponseEntity atualizar(...) { ... }

- public ResponseEntity excluir(@PathVariable String id) { ... }
+ public ResponseEntity excluir(@PathVariable Integer id) { ... }
```
#### Motivo
- **`@PostMapping → @PutMapping`** — usar `POST` no endpoint de atualização gerava conflito de roteamento com o endpoint de cadastro, pois ambos compartilhavam o mesmo caminho base e o mesmo verbo HTTP.
- **`String → Integer` no `@PathVariable`** — o campo `id` da entidade `Aluno` é do tipo `Integer`. Receber o parâmetro como `String` quebrava a chamada de `repository.deleteById(id)` por incompatibilidade de tipos.
---
### 2 · `ProfessorController.java`
**Caminho:** `src/main/java/com/github/app/controller/ProfessorController.java`
#### Alterações
```diff
+ @Transactional
  public ResponseEntity cadastrar(...) { ... }
```
#### Motivo
- A ausência de `@Transactional` deixava a operação de persistência fora de uma transação gerenciada, podendo causar inconsistência no banco em caso de falha.
---
### 3 · `MatriculaController.java`
**Caminho:** `src/main/java/com/github/app/controller/MatriculaController.java`
#### Alterações
```diff
- alunoRepository.getReferenceById(dados.alunoId().toString())
+ alunoRepository.getReferenceById(dados.alunoId())

- matriculaRepository.deleteById(id.toString())
+ matriculaRepository.deleteById(id)
```
#### Motivo
- `AlunoRepository` usa `Integer` como tipo de ID. Converter para `String` via `.toString()` tornava a chamada incompatível com a assinatura do repositório, quebrando tanto a busca por referência quanto a remoção.
---
### 4 · `Aluno.java`
**Caminho:** `src/main/java/com/github/app/model/aluno/Aluno.java`
#### Alterações
```diff
- @Table(name = "professores")
+ @Table(name = "alunos")
```
#### Motivo
- A entidade `Aluno` estava mapeada para a tabela `professores`. Isso fazia com que todas as operações de leitura e escrita de alunos fossem direcionadas à tabela errada, corrompendo os dados.
---
### 5 · `DadosListagemAluno.java`
**Caminho:** `src/main/java/com/github/app/model/aluno/DadosListagemAluno.java`
#### Alterações
```diff
  public DadosListagemAluno(Aluno aluno) {
-     this.nome  = aluno.getEmail();   // ❌ invertido
-     this.email = aluno.getNome();    // ❌ invertido
+     this.nome  = aluno.getNome();    // ✅
+     this.email = aluno.getEmail();   // ✅
  }
```
#### Motivo
- Os campos `nome` e `email` estavam invertidos no construtor do DTO, fazendo com que a API retornasse o JSON com os valores trocados.
---
### 6 · `Professor.java`
**Caminho:** `src/main/java/com/github/app/model/professor/Professor.java`
#### Alterações
```diff
  public void atualizarInformacoes(DadosAtualizacaoProfessor dados) {
      if (dados.nome()  != null) this.nome  = dados.nome();
-     if (dados.email() != null) this.nome  = dados.email();  // ❌ campo errado
+     if (dados.email() != null) this.email = dados.email();  // ✅
  }
```
#### Motivo
- O método sobrescrevia `this.nome` ao tentar atualizar o e-mail, deixando `this.email` inalterado e corrompendo o nome do professor silenciosamente.
---
## 📁 Lista Completa de Arquivos Afetados
<details>
<summary>Clique para expandir</summary>
```
src/main/java/com/github/app/
├── controller/
│   ├── AlunoController.java              ✏️ modificado
│   ├── ProfessorController.java          ✏️ modificado
│   └── MatriculaController.java          ✏️ modificado
│
├── model/
│   ├── aluno/
│   │   ├── Aluno.java                    ✏️ modificado
│   │   ├── AlunoRepository.java          📄 referenciado
│   │   ├── DadosListagemAluno.java       ✏️ modificado
│   │   ├── DadosCadastroAluno.java       📄 referenciado
│   │   └── DadosAtualizacaoAluno.java    📄 referenciado
│   │
│   ├── professor/
│   │   ├── Professor.java                ✏️ modificado
│   │   ├── ProfessorRepository.java      📄 referenciado
│   │   ├── DadosCadastroProfessor.java   📄 referenciado
│   │   ├── DadosAtualizacaoProfessor.java 📄 referenciado
│   │   └── DadosListagemProfessor.java   📄 referenciado
│   │
│   ├── matricula/
│   │   ├── DadosCadastroMatricula.java   📄 referenciado
│   │   └── MatriculaRepository.java      📄 referenciado
│   │
│   └── endereco/
│       ├── Endereco.java                 📄 referenciado
│       └── DadosCadastroEndereco.java    📄 referenciado
```