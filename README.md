# 🏫 Caça aos Erros: Sistema Escolar de Cadastro

**Instituição:** Senac  
**Avaliação:** Projeto Integrador  
**Aluno:** Pedro Costa  

---

# 📋 README — Modificações do Projeto

> Registro das correções aplicadas ao projeto, com descrição objetiva do que foi alterado e o motivo técnico de cada mudança.

---

## 📂 Visão Geral das Alterações

| Alteração | Arquivo | O que foi corrigido |
|:---------:|---------|---------------------|
| 1 | `controller/AlunoController.java` | Verbo HTTP `@PostMapping` → `@PutMapping` no endpoint de atualização |
| 2 | `controller/AlunoController.java` | Tipo do `@PathVariable` de `String` → `Integer` no método `excluir` |
| 3 | `controller/ProfessorController.java` | Adição de `@Transactional` no método `cadastrar` |
| 4 | `controller/MatriculaController.java` | Remoção de `.toString()` no `getReferenceById` do `alunoRepository` |
| 5 | `controller/MatriculaController.java` | Correção do nome do `@PathVariable` de `ids` → `id` no método `excluir` |
| 6 | `model/aluno/Aluno.java` | Mapeamento `@Table(name = "professores")` → `@Table(name = "alunos")` |
| 7 | `model/aluno/AlunoRepository.java` | Tipagem do repositório de `JpaRepository<Aluno, String>` → `JpaRepository<Aluno, Integer>` |
| 8 | `model/aluno/DadosListagemAluno.java` | Campos `nome` e `email` invertidos no construtor do DTO |
| 9 | `model/professor/Professor.java` | Atribuição de `this.nome` → `this.email` no método `atualizarInformacoes` |

---

## 🔧 Detalhamento das Alterações

---

### Alteração 1 · Verbo HTTP incorreto em `AlunoController`

**Arquivo:** `src/main/java/com/github/app/controller/AlunoController.java`  
**Método:** `atualizar(...)`

```diff
- @PostMapping
+ @PutMapping
  @Transactional
  public void atualizar(@RequestBody DadosAtualizacaoAluno dados) { ... }
```

**Motivo:** usar `@PostMapping` no endpoint de atualização gerava conflito direto de roteamento com o endpoint de cadastro, pois ambos compartilhavam o mesmo caminho base e o mesmo verbo HTTP, tornando a rota de atualização inacessível.

---

### Alteração 2 · Tipo do `@PathVariable` incorreto em `AlunoController`

**Arquivo:** `src/main/java/com/github/app/controller/AlunoController.java`  
**Método:** `excluir(...)`

```diff
- public void excluir(@PathVariable String id) {
+ public void excluir(@PathVariable Integer id) {
      repository.deleteById(id);
  }
```

**Motivo:** o campo `id` da entidade `Aluno` é do tipo `Integer`. Receber o parâmetro como `String` causava incompatibilidade de tipos na chamada de `repository.deleteById(id)`, gerando erro em tempo de execução.

---

### Alteração 3 · `@Transactional` ausente em `ProfessorController`

**Arquivo:** `src/main/java/com/github/app/controller/ProfessorController.java`  
**Método:** `cadastrar(...)`

```diff
  @PostMapping
+ @Transactional
  public void cadastrar(@RequestBody DadosCadastroProfessor dados) {
      repository.save(new Professor(dados));
  }
```

**Motivo:** sem `@Transactional`, a operação de `save` era executada fora de um contexto transacional gerenciado pelo Spring, podendo causar falhas silenciosas de persistência em cenários de erro.

---

### Alteração 4 · Conversão desnecessária para `String` em `MatriculaController`

**Arquivo:** `src/main/java/com/github/app/controller/MatriculaController.java`  
**Método:** `cadastrar(...)`

```diff
- Aluno aluno = alunoRepository.getReferenceById(dados.alunoId().toString());
+ Aluno aluno = alunoRepository.getReferenceById(dados.alunoId());
```

**Motivo:** `AlunoRepository` é tipado com `Integer` como chave. Converter o ID para `String` via `.toString()` tornava a chamada incompatível com a assinatura do repositório, quebrando o acesso à referência do aluno.

---

### Alteração 5 · Nome do `@PathVariable` não corresponde à rota em `MatriculaController`

**Arquivo:** `src/main/java/com/github/app/controller/MatriculaController.java`  
**Método:** `excluir(...)`

```diff
  @DeleteMapping("/{id}")
  @Transactional
- public void excluir(@PathVariable Integer ids) {
-     repository.deleteById(ids);
+ public void excluir(@PathVariable Integer id) {
+     repository.deleteById(id);
  }
```

**Motivo:** o nome `ids` não correspondia ao template `{id}` da URL. O Spring não conseguia fazer o binding do parâmetro, causando falha na exclusão de matrículas.

---

### Alteração 6 · Entidade `Aluno` mapeada para tabela errada

**Arquivo:** `src/main/java/com/github/app/model/aluno/Aluno.java`

```diff
  @Entity
- @Table(name = "professores")
+ @Table(name = "alunos")
  public class Aluno { ... }
```

**Motivo:** com o mapeamento errado, todas as operações de leitura e escrita de alunos eram direcionadas à tabela `professores`, corrompendo os dados e misturando registros de entidades diferentes.

---

### Alteração 7 · Tipagem do ID incorreta em `AlunoRepository`

**Arquivo:** `src/main/java/com/github/app/model/aluno/AlunoRepository.java`

```diff
- public interface AlunoRepository extends JpaRepository<Aluno, String> {
+ public interface AlunoRepository extends JpaRepository<Aluno, Integer> {
  }
```

**Motivo:** o campo `id` da entidade `Aluno` é declarado como `Integer`. Usar `String` na assinatura do repositório gerava incompatibilidade de tipos em todas as operações que dependiam do ID, incluindo buscas e exclusões nos controllers.

---

### Alteração 8 · Campos `nome` e `email` invertidos em `DadosListagemAluno`

**Arquivo:** `src/main/java/com/github/app/model/aluno/DadosListagemAluno.java`  
**Método:** construtor `DadosListagemAluno(Aluno aluno)`

```diff
  public DadosListagemAluno(Aluno aluno) {
-     this.nome  = aluno.getEmail();   // ❌ invertido
-     this.email = aluno.getNome();    // ❌ invertido
+     this.nome  = aluno.getNome();    // ✅
+     this.email = aluno.getEmail();   // ✅
  }
```

**Motivo:** os campos estavam trocados no construtor do DTO, fazendo com que a API retornasse o JSON com `nome` e `email` com os valores invertidos para todos os alunos listados.

---

### Alteração 9 · Atribuição no campo errado em `Professor`

**Arquivo:** `src/main/java/com/github/app/model/professor/Professor.java`  
**Método:** `atualizarInformacoes(DadosAtualizacaoProfessor dados)`

```diff
  public void atualizarInformacoes(DadosAtualizacaoProfessor dados) {
      if (dados.nome()  != null) this.nome  = dados.nome();
-     if (dados.email() != null) this.nome  = dados.email();  // ❌ campo errado
+     if (dados.email() != null) this.email = dados.email();  // ✅
  }
```

**Motivo:** ao atualizar o e-mail, o código sobrescrevia `this.nome` com o valor do e-mail, corrompendo o nome do professor silenciosamente e deixando o campo `this.email` sempre inalterado.

---

## 📁 Lista Completa de Arquivos Afetados

<details>
<summary>Clique para expandir</summary>

```
src/main/java/com/github/app/
├── controller/
│   ├── AlunoController.java              ✏️ modificado  (alterações 1, 2)
│   ├── ProfessorController.java          ✏️ modificado  (alteração 3)
│   └── MatriculaController.java          ✏️ modificado  (alterações 4, 5)
│
├── model/
│   ├── aluno/
│   │   ├── Aluno.java                    ✏️ modificado  (alteração 6)
│   │   ├── AlunoRepository.java          ✏️ modificado  (alteração 7)
│   │   ├── DadosListagemAluno.java       ✏️ modificado  (alteração 8)
│   │   ├── DadosCadastroAluno.java       📄 referenciado
│   │   └── DadosAtualizacaoAluno.java    📄 referenciado
│   │
│   ├── professor/
│   │   ├── Professor.java                ✏️ modificado  (alteração 9)
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

</details>

---

## ▶️ Como Rodar o Projeto

> Após aplicar as modificações, siga um dos métodos abaixo para iniciar a aplicação.

---

### ✅ Pré-requisitos

Antes de rodar, certifique-se de ter instalado:

| Ferramenta | Versão recomendada |
|------------|--------------------|
| Java (JDK) | 17 ou superior |
| Maven | 3.8 ou superior |
| VS Code | Versão atual |
| Extensão [Spring Boot Extension Pack](https://marketplace.visualstudio.com/items?itemName=vmware.vscode-boot-dev-pack) | Instalada |

---

### Método 1 — Pelo VS Code (recomendado)

1. Abra o projeto no VS Code (`File → Open Folder`)
2. Aguarde o VS Code indexar o projeto (barra inferior ficará estável)
3. Navegue até o arquivo:
   ```
   src/main/java/com/github/app/AppApplication.java
   ```
4. Com o arquivo aberto, clique no botão **▶ Run** que aparece acima do método `main`, ou pressione:
   ```
   Ctrl + F5
   ```
5. Acompanhe o log no terminal integrado — a aplicação estará pronta quando aparecer:
   ```
   Started AppApplication in X.XXX seconds
   ```

> 💡 **Dica:** o painel **Spring Boot Dashboard** (ícone de folha na barra lateral) lista a aplicação e permite iniciar/parar com um clique.

---

### Método 2 — Pelo Terminal Integrado (Maven)

Abra o terminal no VS Code (`` Ctrl + ` ``) e execute:

```bash
# Compilar e iniciar
./mvnw spring-boot:run
```

> No Windows, se `./mvnw` não funcionar, use:
> ```cmd
> mvnw.cmd spring-boot:run
> ```

---

### Método 3 — Build + execução do JAR

Use este método para gerar um artefato final ou simular ambiente de produção:

```bash
# 1. Gerar o JAR (pula os testes para agilizar)
./mvnw clean package -DskipTests

# 2. Executar o JAR gerado
java -jar target/*.jar
```

---

### 🛑 Parando a aplicação

| Ambiente | Como parar |
|----------|------------|
| VS Code (Run) | Clique no ícone ⏹ no painel de debug ou no Spring Boot Dashboard |
| Terminal | `Ctrl + C` |

---

### ⚠️ Problemas comuns

| Sintoma | Causa provável | Solução |
|---------|---------------|---------|
| Porta `8080` já em uso | Outra instância rodando | `Ctrl+C` no terminal anterior ou mude a porta em `application.properties`: `server.port=8081` |

---
