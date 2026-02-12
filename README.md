
# 🛡️ Trabalho T2.2 — Testes de Segurança em Aplicação Web Vulnerável

Nomes: Guilherme Soares, Pedro Rosa e Victor Setubal
Disciplina: Segurança da Informação
Professor: NOME DO PROFESSOR
Data: 2026

## 🎯 Objetivo

Este trabalho tem como objetivo configurar um ambiente vulnerável e realizar testes de segurança utilizando ferramentas automáticas e técnicas manuais, identificando vulnerabilidades reais e propondo medidas de mitigação.

##🖥️ Ambiente Utilizado

Sistema Operacional: Windows 10 / Linux / Outro

Docker: versão XX

Aplicação vulnerável: OWASP Juice Shop

Navegador: Google Chrome

Ferramentas de teste:

OWASP ZAP

sqlmap

Burp Suite Community

##📦 Instalação do Ambiente Vulnerável
###1️⃣ Instalar Docker

Baixar e instalar:

https://www.docker.com/products/docker-desktop/

Verificar instalação:

'''docker --version'''

###2️⃣ Baixar OWASP Juice Shop
'''docker pull bkimminich/juice-shop'''

###3️⃣ Executar o servidor vulnerável
'''docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop'''

###4️⃣ Acessar aplicação

Abrir no navegador:

'''http://localhost:3000'''

###5️⃣ Parar e iniciar o container

Parar:

'''docker stop juice-shop'''


Iniciar novamente:

'''docker start juice-shop'''


Remover:

'''docker rm -f juice-shop'''

##🔎 Ferramentas Utilizadas
OWASP ZAP

Ferramenta para análise automática de vulnerabilidades web.

Download:
https://www.zaproxy.org/download/

sqlmap

Ferramenta para exploração automática de SQL Injection.

Download:
https://sqlmap.org/

Burp Suite Community

Interceptação de requisições HTTP.

Download:
https://portswigger.net/burp/communitydownload

##🧪 Procedimento Experimental
###1️⃣ Execução do OWASP ZAP

Abrir OWASP ZAP

Selecionar "Automated Scan"

Inserir URL:

'''http://localhost:3000'''


Executar varredura

Registrar vulnerabilidades encontradas

Prints salvos em:

'''/prints/zap'''

###2️⃣ Teste de SQL Injection (Manual)
Objetivo

Bypass de autenticação utilizando injeção SQL.

Procedimento

Acessar tela de login

Inserir no campo email:

''' ' OR 1=1 -- '''


Senha: qualquer valor

Resultado esperado

Acesso ao sistema sem credenciais válidas.

Explicação técnica

A aplicação não valida corretamente os dados de entrada, permitindo a modificação da query SQL.

Exemplo de query vulnerável:

'''SELECT * FROM users WHERE email = '' OR 1=1 -- ' AND password='123';'''


O trecho 1=1 sempre é verdadeiro.

###3️⃣ Teste de XSS (Cross-Site Scripting)
Procedimento

Inserir em campo de busca ou comentário:

'''<script>alert('XSS')</script>'''

Resultado esperado

Execução do script no navegador.

Impacto

Roubo de sessão

Execução de código malicioso

Redirecionamento do usuário

##🧾 Vulnerabilidades Encontradas
SQL Injection

Tipo: Injection

Severidade: Alta

Impacto: acesso indevido ao sistema

Cross-Site Scripting (XSS)

Tipo: Client-side attack

Severidade: Média

Impacto: execução de scripts maliciosos

Headers inseguros

Falta de CSP

Falta de X-Frame-Options

##🔐 Mitigações Propostas
Para SQL Injection

Uso de Prepared Statements

Validação de entradas

Uso de ORM seguro

Para XSS

Escape de saída

Sanitização de dados

Implementação de Content Security Policy (CSP)

##📸 Evidências

As evidências estão na pasta:

'''/prints'''


Contendo:

- Execução do ZAP

- Exploração SQL Injection

- Exploração XSS

- Requisições HTTP interceptadas
