# Sample Flask Auth

> API de autenticação e gerenciamento de usuários construída com Flask, Flask-Login e MySQL

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-2.3.0-000000?style=flat&logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![Flask--SQLAlchemy](https://img.shields.io/badge/Flask--SQLAlchemy-3.1.1-D71F00?style=flat)](https://flask-sqlalchemy.palletsprojects.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=flat&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)

---

## 📋 Sobre o projeto

**Sample Flask Auth** é uma API REST de autenticação e gerenciamento de usuários, desenvolvida como projeto de estudo. O objetivo é praticar autenticação baseada em sessão com **Flask-Login**, persistência de dados com **Flask-SQLAlchemy** sobre **MySQL**, hash seguro de senhas com **bcrypt** e controle de acesso baseado em papéis (roles).

O banco de dados MySQL é provisionado via **Docker Compose**, facilitando a configuração do ambiente local.

### Principais características

- ✅ **Autenticação por sessão** com Flask-Login (`login`/`logout`)
- ✅ **Senhas com hash seguro** usando `bcrypt`
- ✅ **CRUD de usuários** — criação, consulta, atualização e remoção
- ✅ **Controle de acesso por papel (role)** — usuários comuns (`user`) e administradores (`admin`)
- ✅ **Rotas protegidas** com o decorator `@login_required`
- ✅ **Banco de dados MySQL** via Docker Compose

---

## 🏗️ Arquitetura

### Stack utilizada

- **Linguagem**: Python 3.10+
- **Framework Web**: Flask 2.3.0
- **ORM**: Flask-SQLAlchemy 3.1.1
- **Autenticação**: Flask-Login 0.6.2
- **Banco de dados**: MySQL (via `pymysql` 1.1.0)
- **Hash de senha**: bcrypt 4.1.2
- **Infraestrutura local**: Docker Compose

### Estrutura do projeto

```
sample-flask-auth/
├── models/
│   └── user.py         # Modelo User (SQLAlchemy)
├── app.py               # Aplicação Flask, rotas e autenticação
├── database.py            # Instância do SQLAlchemy
├── docker-compose.yml       # Container MySQL para desenvolvimento
├── requirements.txt         # Dependências do projeto
└── .gitignore
```

### Modelo de dados

O modelo `User` (`models/user.py`) representa os usuários da aplicação, com pelo menos os seguintes campos:

| Campo      | Tipo   | Descrição                                      |
|------------|--------|--------------------------------------------------|
| `id`       | int    | Identificador único do usuário                   |
| `username` | string | Nome de usuário, usado no login                  |
| `password` | string | Hash da senha (gerado com bcrypt)                |
| `role`     | string | Papel do usuário (`user` ou `admin`)              |

---

## 🚀 Como executar

### Pré-requisitos

- Python 3.10 ou superior
- pip
- Docker e Docker Compose (para o banco de dados MySQL)

### Instalação

**1. Clone o repositório**

```bash
git clone https://github.com/igorland1/sample-flask-auth.git
cd sample-flask-auth
```

**2. Instale as dependências**

```bash
pip install -r requirements.txt
```

**3. Suba o banco de dados MySQL**

```bash
docker compose up -d
```

> ⚠️ O `docker-compose.yml` original define um volume apontando para um caminho local do Windows (`C:/Users/igorlandi/mysql`). Ajuste esse caminho no arquivo conforme o seu ambiente antes de subir o container.

O banco `flask-crud` será criado automaticamente, acessível em `127.0.0.1:3306` com usuário `admin` e senha `admin123` (credenciais definidas para desenvolvimento local).

**5. Configure a conexão com o banco**

A string de conexão está definida diretamente em `app.py`:

```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://admin:admin123@127.0.0.1:3306/flask-crud'
```

> 💡 Para uso além do ambiente local, recomenda-se mover essa configuração (e a `SECRET_KEY`) para variáveis de ambiente.

**6. Execute a aplicação**

```bash
python app.py
```

A API estará disponível em `http://127.0.0.1:5000`, em modo debug.

---

## 📡 Endpoints da API

### Visão geral

```http
POST    /login              # Autentica um usuário e inicia a sessão
GET     /logout             # Encerra a sessão do usuário autenticado
POST    /user               # Cria um novo usuário
GET     /user/<id_user>     # Consulta um usuário pelo ID (requer login)
PUT     /user/<id_user>     # Atualiza a senha de um usuário (requer login)
DELETE  /user/<id_user>     # Remove um usuário (requer permissão de admin)
```

### Regras de acesso

- As rotas `/user/<id_user>` (GET, PUT, DELETE) e `/logout` exigem sessão ativa (`@login_required`).
- Na atualização (`PUT`), um usuário com papel `user` só pode alterar a própria senha.
- Na remoção (`DELETE`), apenas usuários com papel `admin` podem excluir contas, e não é permitido que um admin exclua a própria conta.

### Exemplos de requisições

**Criar usuário**

```http
POST /user
Content-Type: application/json

{
  "username": "igor",
  "password": "minhasenha123"
}
```

Resposta:

```json
{
  "message": "Usuário cadastrado com sucesso"
}
```

**Login**

```http
POST /login
Content-Type: application/json

{
  "username": "igor",
  "password": "minhasenha123"
}
```

Resposta:

```json
{
  "message": "Autenticação realizada com sucesso"
}
```

**Consultar usuário**

```http
GET /user/1
```

Resposta:

```json
{
  "username": "igor"
}
```

**Atualizar senha**

```http
PUT /user/1
Content-Type: application/json

{
  "password": "novaSenha456"
}
```

**Remover usuário** *(apenas admin)*

```http
DELETE /user/2
```

**Logout**

```http
GET /logout
```

---

## 🔒 Segurança

- **Hash de senhas** com `bcrypt`, nunca armazenadas em texto puro
- **Sessão de usuário** gerenciada pelo Flask-Login
- **Controle de acesso por papel** (`user` / `admin`) nas rotas sensíveis
- **Proteções contra autoexclusão** de contas administrativas

> ⚠️ Este projeto tem fins de estudo: a `SECRET_KEY` e as credenciais do banco estão hardcoded em `app.py`/`docker-compose.yml` para simplificar o ambiente local. Em um cenário real, essas informações devem vir de variáveis de ambiente e nunca ser versionadas.

---

## 🎯 Aprendizados demonstrados neste projeto

- Autenticação baseada em sessão com Flask-Login
- Hash e verificação segura de senhas com bcrypt
- Modelagem de dados com Flask-SQLAlchemy e integração com MySQL
- Controle de acesso baseado em papéis (RBAC simplificado)
- Uso de Docker Compose para provisionar dependências de desenvolvimento

---

## 📝 Licença

Projeto de estudo, disponibilizado como código aberto para fins de aprendizado e portfólio.

---

## 🤝 Contato

Projeto desenvolvido por [Igor](https://github.com/igorland1).