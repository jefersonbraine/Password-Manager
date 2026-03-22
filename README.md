# Gerenciador de Senhas (Python)

Um gerenciador de senhas simples em linha de comando (CLI), desenvolvido em Python, com criptografia simétrica usando **Fernet** (`cryptography`).

O projeto permite:

- salvar senhas associadas a um domínio,
- criptografar os dados antes de persistir,
- recuperar a senha descriptografada com a chave correta.

## Visao Geral

Este projeto foi organizado em uma estrutura parecida com MVC:

- **model**: persistencia de dados em arquivo texto,
- **views**: logica de criptografia e descriptografia,
- **templates**: fluxo de interacao com o usuario no terminal.

A ideia principal e manter o projeto didatico e objetivo, sem banco de dados externo.

## Funcionalidades

- Cadastro de novas senhas por dominio.
- Criptografia da senha com Fernet antes de salvar.
- Geracao de chave na primeira execucao de cadastro.
- Opcao de arquivar a chave em arquivo na pasta `keys/`.
- Consulta de senha por dominio (com chave informada pelo usuario).
- Persistencia local em arquivo texto (`db/Password.txt`).

## Arquitetura do Projeto

```text
Gerenciador-de-senhas/
├── db/
│   └── Password.txt            # Base de dados local (texto)
├── keys/
│   └── key.key                 # Chave Fernet gerada
├── model/
│   └── password.py             # Modelos e persistencia
├── templates/
│   └── template.py             # Script principal (CLI)
├── views/
│   └── password_views.py       # Criptografia/descriptografia
└── venv/                       # Ambiente virtual local
```

### Responsabilidade de cada modulo

- `model/password.py`
  - `BaseModel.save()`: salva objetos em arquivo texto separado por `|`.
  - `BaseModel.get()`: le arquivo e reconstrui registros em lista de dicionarios.
  - `Password`: representa um registro com `domain`, `password`, `create_at`.

- `views/password_views.py`
  - `FernetHasher.create_key()`: gera chave baseada em string aleatoria + SHA-256 + Base64.
  - `FernetHasher.encrypt()`: criptografa valor com Fernet.
  - `FernetHasher.decrypt()`: descriptografa valor e trata token invalido.

- `templates/template.py`
  - Interface em terminal para:
    - salvar nova senha,
    - consultar senha existente.

## Tecnologias

- Python 3.12 (detectado no `venv` do projeto)
- Biblioteca `cryptography` (Fernet)
- Armazenamento em arquivo `.txt`

## Como Executar

### 1) Pre-requisitos

- Python 3.10+ (recomendado: 3.12)
- `pip`

### 2) Criar e ativar ambiente virtual

### Windows (PowerShell)

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### Linux/macOS

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3) Instalar dependencias

```bash
pip install cryptography
```

### 4) Executar o sistema

A partir da raiz do projeto:

```bash
python templates/template.py
```

## Fluxo de Uso

Ao iniciar, o sistema pede:

```text
Digite 1 para salvar uma nova senha ou 2 para ver uma senha salva
```

### Opcao 1: Salvar nova senha

- Se nao houver registros em `db/Password.txt`, uma nova chave e criada.
- O sistema exibe a chave e pode salvar em `keys/`.
- Usuario informa `dominio` e `senha`.
- Senha e criptografada e persistida no arquivo.

### Opcao 2: Ver senha salva

- Usuario informa dominio e chave.
- O sistema busca no arquivo e tenta descriptografar.
- Se a chave estiver correta, exibe a senha em texto plano.

## Formato dos Dados

Cada linha em `db/Password.txt` segue o formato:

```text
domain|password_criptografada|create_at
```

Exemplo:

```text
instagram|gAAAAAB...|2024-11-01T22:31:07.651267
```

## Seguranca e Boas Praticas

- Nao versione chaves reais (`keys/*.key`) em repositorios publicos.
- Nao versione base real de senhas (`db/Password.txt`) com dados sensiveis.
- Armazene chave em local seguro (gerenciador de segredos/cofre).
- A mesma chave deve ser usada para descriptografar os dados existentes.
- Se a chave for perdida, nao ha como recuperar as senhas criptografadas.

Sugestao: adicionar `.gitignore` para ignorar:

- `venv/`
- `keys/*.key`
- `db/Password.txt`
- `__pycache__/`

## Limites Atuais do Projeto

- Nao ha autenticacao de usuario mestre.
- Persistencia em arquivo texto (sem banco e sem controle de concorrencia).
- Consulta por dominio usa comparacao parcial (`if domain in i['domain']`).
- Nao existe edicao/remocao de senha.
- Nao ha testes automatizados.
- Em alguns cenarios de consulta sem resultado, o script pode gerar erro por variavel nao inicializada (`password`).

## Melhorias Recomendadas

- Criar `requirements.txt` e `README` com instrucoes de setup automatizado.
- Implementar senha mestra + derivacao de chave com PBKDF2/Argon2.
- Trocar armazenamento em texto por SQLite.
- Implementar comandos: listar, editar, excluir, rotacionar chave.
- Criar testes unitarios para criptografia e persistencia.
- Tratar validacoes e erros de entrada de forma mais robusta.

## Possivel Roadmap

1. Refatorar CLI para `argparse`.
2. Adicionar camada de servico (casos de uso).
3. Implementar testes com `pytest`.
4. Migrar para banco de dados local.
5. Empacotar como ferramenta instalavel (`pip`).

## Licenca

Este repositorio nao possui arquivo de licenca definido ate o momento.

Se quiser distribuir ou reutilizar o projeto, recomenda-se adicionar um arquivo `LICENSE` (por exemplo: MIT).
