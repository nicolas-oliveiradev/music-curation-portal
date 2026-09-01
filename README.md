# 🎵 Music Curation & Analytics Portal

Um portal de curadoria e gerenciamento de dados musicais desenvolvido para simular um ecossistema interno de catálogo e análise de faixas. A aplicação consome uma base de dados relacional (PostgreSQL) e oferece uma interface interativa no Appsmith para operações completas de CRUD com arquitetura de validação resiliente em duas camadas.

---
## 🖼️ Interface Atual e Validações

### Dashboard & Tabela de Músicas
[Tabela Principal e Edição Inline](LINK_DO_PRINT_1)
> *Tabela dinâmica vinculada ao PostgreSQL com suporte a ordenação, busca e edição direta de células.*

### Validação de Dados em Camadas (Fail-Fast)
[Erro de Validação na Interface](LINK_DO_PRINT_2)
> *O próprio Appsmith impede a submissão de notas fora do intervalo 1-5, exibindo alerta antes da requisição ao banco.*
---
## 🚀 Tecnologias Utilizadas

* **Frontend & Dashboard:** Appsmith (Low-Code Enterprise Framework)
* **Lógica & Eventos:** JavaScript (ES6+ / Promises / Event-Driven Architecture)
* **Banco de Dados:** PostgreSQL (Supabase / Nuvem)
* **Linguagem de Consultas:** SQL Dinâmico (`COALESCE`, `NULLIF`, `WHERE` condicional)

---

## 🛠️ Arquitetura e Destaques Técnicos

### 1. Validação Resiliente em Camadas (*Defense in Depth*)
Para garantir a integridade das informações e prevenir falhas de gravação em produção, o sistema conta com dupla validação:
* **Client-Side (Appsmith):** Regras de validação nativas na interface (limitação de intervalo de 1 a 5 para notas e checagem de tipos numéricos). Bloqueia requisições inválidas antes que cheguem à rede, economizando recursos e fornecendo *feedback* instantâneo (*Fail-Fast*).
* **Server-Side (PostgreSQL):** Sanitização de inputs através de SQL dinâmico e regras de restrição do próprio banco de dados (*Constraints*).

### 2. Edição Atômica e Tratamento de Nulos
A query de `UPDATE` foi projetada para suportar edições parciais por célula (*inline editing*) sem corromper atributos não editados da linha:

```sql
UPDATE musicas
SET 
  titulo = COALESCE(NULLIF({{ Table1.updatedRow.titulo }}, ''), titulo),
  duracao_segundos = COALESCE({{ Table1.updatedRow.duracao_segundos ?? null }}, duracao_segundos),
  nota_pessoal = COALESCE({{ Table1.updatedRow.nota_pessoal ?? null }}, nota_pessoal)
WHERE id = {{ Table1.updatedRow.id ?? Table1.selectedRow.id }};

```
`COALESCE` (SQL): Preserva o valor original do banco caso o campo enviado venha como NULL.

`NULLIF` (SQL): Converte strings vazias em NULL, evitando gravações em branco acidentais.

`??` null (JavaScript): Converte estados undefined (campos não alterados na célula) em nulos explícitos para o driver do PostgreSQL.

### 3. Gerenciamento Assíncrono de Estado
Integração de Promises JS nos manipuladores de eventos da tabela (update_musica.run()).

Encadeamento de chamadas para recarregar o dataset dinamicamente (get_musics.run()).

Feedback visual contextualizado para o usuário via notificações (showAlert).

## 📌 Funcionalidades (CRUD)

| Operação | Componente UI | Mecanismo Backend |
| :--- | :--- | :--- |
| **Create (Inserir)** | Formulário `Form1` | Query SQL `INSERT` com captura de inputs |
| **Read (Listar)** | Tabela Dinâmica `Table1` | Query SQL `SELECT` ordenada com renderização em tempo real |
| **Update (Editar)** | Células Editáveis (*Inline*) | Query SQL `UPDATE` parametrizada disparada no evento da coluna |
| **Delete (Excluir)** | Botão de Ação por Linha | Query SQL `DELETE` parametrizada por `id` |

## ⚙️ Como Executar / Importar o Projeto
Clone este repositório ou faça o download do arquivo de backup JSON da aplicação.

Acesse a sua conta no Appsmith.

No painel principal, clique em Import e selecione o arquivo JSON exportado.

Configure a sua conexão de Datasource apontando para a sua instância do PostgreSQL.

Certifique-se de que a tabela `musicas` contenha a seguinte estrutura básica:

```sql
CREATE TABLE musicas (
    id SERIAL PRIMARY KEY,
    titulo VARCHAR(255) NOT NULL,
    artista VARCHAR(255),
    genero VARCHAR(100),
    duracao_segundos INT,
    nota_pessoal INT CHECK (nota_pessoal BETWEEN 1 AND 5),
    favorita BOOLEAN DEFAULT FALSE
);

```
## ✒️ Autor
# Nicolas Oliveira

Estudante de Engenharia de Software | Desenvolvedor com foco em Software Engineering, Análise de Dados e Soluções Web.
