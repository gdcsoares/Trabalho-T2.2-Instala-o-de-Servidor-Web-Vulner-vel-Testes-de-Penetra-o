
# 🛡️ Trabalho T2.2 — Testes de Segurança em Aplicação Web Vulnerável

- Nomes: Guilherme Soares, Pedro Rosa e Victor Setubal

- Disciplina: Segurança da Informação

- Professor: Rodolfo

- Data: 2026
  
--- 

## 🎯 Objetivo

Este trabalho tem como objetivo configurar um ambiente vulnerável e realizar testes de segurança utilizando ferramentas automáticas e técnicas manuais, identificando vulnerabilidades reais e propondo medidas de mitigação.

---

## 🖥️ Ambiente Utilizado

- Sistema Operacional: Linux 

- Docker: versão 26.1.3

- Aplicação vulnerável: OWASP Juice Shop

- Ferramentas de teste:

  - OWASP ZAP

  - sqlmap


---

## 📦 Instalação e Execução do Ambiente Vulnerável
### 1️⃣ Instalar Docker

Baixar e instalar:

https://www.docker.com/products/docker-desktop/

Verificar instalação:

``` docker --version ```

---

### 2️⃣ Baixar OWASP Juice Shop
```docker pull bkimminich/juice-shop```

---

### 3️⃣ Executar o servidor vulnerável
```docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop```

---

### 4️⃣ Acessar aplicação

Abrir no navegador:

```http://localhost:3000```

---

### 5️⃣ Parar e iniciar o container

Parar:

```docker stop juice-shop```

Iniciar novamente:

```docker start juice-shop```


Remover:

```docker rm -f juice-shop```

---

## 🔎 Ferramentas Utilizadas
### OWASP ZAP

Ferramenta para análise automática de vulnerabilidades web.

Download:
https://www.zaproxy.org/download/

---

### sqlmap

Ferramenta para exploração automática de SQL Injection.

Download:
https://sqlmap.org/

---

## 🧪 Procedimento Experimental
### 1️⃣ Execução do OWASP ZAP

1- Abrir OWASP ZAP

2- Selecionar "Automated Scan"

3- Inserir URL:

```http://localhost:3000```


4- Executar varredura

5- Registrar vulnerabilidades encontradas

Prints salvos em:

```/prints/zap```

---

### 2️⃣ Teste de SQL Injection (Manual)
### Objetivo

Bypass de autenticação utilizando injeção SQL.

### Procedimento

1- Acessar tela de login

2- Inserir no campo email:

``` ' OR 1=1 -- ```


3- Senha: qualquer valor

### Resultado esperado

Acesso ao sistema sem credenciais válidas.

### Explicação técnica

A aplicação não valida corretamente os dados de entrada, permitindo a modificação da query SQL.

Exemplo de query vulnerável:

```SELECT * FROM users WHERE email = '' OR 1=1 -- ' AND password='123';```


O trecho 1=1 sempre é verdadeiro e, dessa forma, o login é realizado.

---

### 3️⃣ Teste de usando o sqlmap (SQL Injection automática)
Com o uso do OWASP ZAP foi possível verificar a possibilidade de vulnerabilidade para SQL Injection, qual o URL vulnerável e que o banco de dados é SQLite. Com isso podemos usar o seguinte comando para o sqlmap:

```python3 sqlmap.py -u "http://localhost:3000/rest/products/search?q=test" --dbms=sqlite --prefix "'))" --suffix "--" --level=5 --risk=3 --batch --dump```

Aqui temos que:

- ```-u "http://localhost:3000/rest/products/search?q=test"```: Define a URL alvo. O sqlmap vai focar no parâmetro q (onde está o valor test) para tentar injetar códigos maliciosos. Essa URL vulnerável foi encontrada por meio do OWASP ZAP.
- ```--dbms=sqlite```: Define-se que o alvo utiliza SQLite (descoberto por meio do OWASP ZAP).
- ```-suffix "--"```: Adiciona isso depois do comando de ataque. O -- em SQL serve para comentar o restante da linha original, evitando que erros de sintaxe quebrem o ataque.
- ```--level=5```: O nível máximo de testes (1 a 5). Faz com que o SQLMap tente injetar em mais lugares (como cabeçalhos HTTP, cookies, etc.) e use muito mais payloads.
- ```--risk=3```: O nível máximo de risco (1 a 3). No nível 3, a ferramenta tenta comandos que podem causar danos ou alterações no banco de dados (como queries baseadas em OR). É necessário atenção pois isso pode corromper dados do ambiente.
- ```--batch```: Modo automático. O SQLMap não vai te perguntar "[Y/n]"; ele escolherá sempre a resposta padrão.
- ```--dump```: O objetivo final. Uma vez que ele confirme a vulnerabilidade, ele vai baixar (extrair) todo o conteúdo das tabelas que conseguir acessar.

Quando executamos isso no nosso teste, foi possível descobrir as tabelas existentes no banco de dados. Foi possível descobrir a tabela Users e, por isso, vamos focar nela para tentarmos descobrir credenciais. Para isso, executa-se o seguinte comando: 

```python3 sqlmap.py -u "http://localhost:3000/rest/products/search?q=test" --dbms=sqlite --prefix "'))" --suffix "--" --level=5 --risk=3 -T users --columns```

- ```-T Users```: Define a tabela alvo.

Com isso, conseguimos descobrir as colunas da tabela Users: id, email, password e role. Então podemos especificar essas colunas usando o comando:

```python3 sqlmap.py -u "http://localhost:3000/rest/products/search?q=test" --dbms=sqlite --prefix "'))" --suffix "--" -T Users -C "id,email,password,role" --dump --batch```

onde:

- ```-C "id,email,password,role"```: As colunas de interesse (Descobertos pelo comando do sqlmap utilizado anteriormente).

Com isso, conseguimos obter os usuários, seus papéis (roles) e as senhas (as quais utilizavam para criptografica Hash MD5) sendo que algumas senhas já foram descriptografadas pelo próprio sqlmap.

Com isso, com ajuda do sqlmap e o uso do OWASP ZAP foi possível expor dados que, obviamente, deveriam ser confidenciais.

#### Remediação a esse tipo de ataque:
a) Consultas Parametrizadas (Prepared Statements): Em vez de montar a query como uma string:
- Errado (Vulnerável): "SELECT * FROM Products WHERE name = '" + input + "'"
- Certo (Seguro): Use placeholders (?). O banco de dados tratará o input apenas como texto, e não como parte do comando SQL.

b) Hashing de Senhas Moderno: Nunca use MD5 ou SHA1. Eles são rápidos demais, o que facilita ataques de força bruta. Use Argon2, bcrypt ou scrypt com um "salt" único para cada usuário.

c) Princípio do Menor Privilégio: O usuário que o site usa para acessar o banco de dados não deve ter permissão para ver tabelas de sistema ou deletar dados. Ele deve ter acesso apenas ao estritamente necessário.

d) Web Application Firewall (WAF): Um WAF poderia detectar e bloquear o tráfego do sqlmap ao identificar padrões comuns de ataque (como o uso excessivo de UNION SELECT ou ORDER BY).

---

### 4️⃣ Teste de XSS (Cross-Site Scripting)
### Procedimento

Inserir em campo de busca ou comentário:

```<img src=x onerror=alert('XSS')>```

### Resultado esperado

Execução do script no navegador.

### Impacto

- Roubo de sessão

- Execução de código malicioso

- Redirecionamento do usuário

---

## 🧾 Vulnerabilidades Encontradas
### SQL Injection

- Tipo: Injection

- Severidade: Alta

- Impacto: acesso indevido ao sistema

### Cross-Site Scripting (XSS)

- Tipo: Client-side attack

- Severidade: Média

- Impacto: execução de scripts maliciosos

### Headers inseguros

- Falta de CSP

- Falta de X-Frame-Options

---

## 🔐 Mitigações Propostas
### Para SQL Injection

- Uso de Prepared Statements

- Validação de entradas

- Uso de ORM seguro

### Para XSS

Escape de saída

Sanitização de dados

Implementação de Content Security Policy (CSP)

---

## 📸 Evidências

As evidências estão na pasta:

```/prints```


Contendo:

- Execução do ZAP

- Exploração SQL Injection

- Exploração XSS
