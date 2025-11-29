# Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux
Desafio DIO - Implementar, documentar e compartilhar um projeto prático utilizando Kali Linux e a ferramenta Medusa, em conjunto com ambientes vulneráveis 

1. Configuração do Ambiente
Utilização do Kali Linux como sistema principal para execução dos testes.
Preparação de ambientes vulneráveis (máquina virtual metasploitable 2).

2. Ferramenta Utilizada: Medusa
Medusa foi escolhida por ser uma ferramenta de força bruta paralela voltada para testes de autenticação.
Configuração de parâmetros básicos: alvo, serviço, lista de usuários e senhas.

# Validado comunicação com a máquina vulnerável
ping -c 192.168.56.101

# Verifica se as portas listadas estão abertas, fechadas ou filtradas. Caso abertas, tenta identificar o serviço e versão rodando nelas
nmap -sV -p 21,22,80,445,139 192.168.56.101 

- nmap → Ferramenta de varredura de rede.
- -sV → Ativa a detecção de versão dos serviços. Ou seja, além de verificar se a porta está aberta, tenta identificar qual software e versão está rodando (ex.: Apache 2.4, OpenSSH 8.2).
- -p 21,22,80,445,139 → Define quais portas serão analisadas:
- 21 → FTP
- 22 → SSH
- 80 → HTTP (web)
- 445 → SMB (compartilhamento de arquivos no Windows)
- 139 → NetBIOS (antigo protocolo de compartilhamento no Windows)
- 192.168.56.101 → IP alvo dentro da rede local 

# Tentativa de conexão com ftp
ftp 192.168.56.101

# criação do arquivo texto com uma lista de nomes de usuários (um por linha: quebra de linha > \n)
echo -e '\user\nmsfadmin\nadmin\nroot' > users.txt

# criação do arquivo texto com uma lista de senhas (um por linha: quebra de linha > \n)
echo -e '123456\npassword\nqwerty\nmsfadmin' > pass.txt

# ataque de força bruta controlado usando a ferramenta Medusa contra o serviço FTP da máquina alvo .
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6 

- medusa → ferramenta de testes de autenticação em paralelo.
- -h 192.168.56.101 → define o host alvo (IP da máquina vulnerável).
- -U users.txt → arquivo com a lista de usuários a serem testados.
- -P pass.txt → arquivo com a lista de senhas a serem testadas.
- -M ftp → módulo/protocolo escolhido: neste caso, FTP.
- -t 6 → número de threads (processos paralelos). Isso significa que o Medusa vai tentar até 6 combinações simultaneamente, acelerando o ataque.

O Medusa conecta ao serviço FTP do host 192.168.56.101.
Para cada usuário em users.txt, ele testa todas as senhas em pass.txt.
Faz isso em paralelo (até 6 tentativas simultâneas).
Se alguma combinação for válida, retorna o login e senha encontrados.

# ataque de força bruta contra um formulário de login HTTP, especificamente o da aplicação DVWA (Damn Vulnerable Web Application) hospedada no alvo
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE: '/dvwa/login.php' 
-m FORM: 'username="USER"&password="PASS"&login=Login' 
-m 'FAIL-Login failed' -t 6

- -h 192.168.56.101 → IP do host alvo.
- -U users.txt → arquivo com lista de usuários.
- -P pass.txt → arquivo com lista de senhas.
- -M http → módulo HTTP (para atacar formulários web).
- -m PAGE: '/dvwa/login.php' → página alvo do login (endpoint do formulário).
- -m FORM: 'username="USER"&password="PASS"&login=Login' → estrutura do formulário.
- O Medusa substitui USER e PASS pelas combinações dos arquivos users.txt e pass.txt.
- -m 'FAIL-Login failed' → string que indica falha de login. Se essa mensagem aparecer na resposta, significa que a tentativa não funcionou.
- -t 6 → número de threads (6 tentativas simultâneas).

- O Medusa acessa a página /dvwa/login.php no alvo.
- Para cada usuário e senha, envia uma requisição HTTP simulando o login.
- Se a resposta não contiver a string "FAIL-Login failed", significa que a combinação foi aceita.
- O processo é feito em paralelo com até 6 threads, acelerando a execução.

# utilizando a ferramenta enum4linux para coletar informações de um host Windows ou Samba
enum4linux - a 192.168.56.101 | tee enum4_output.txt   

- enum4linux → ferramenta usada para enumeração de informações em sistemas Windows/Samba, muito útil em testes de segurança.
- -a → opção que significa "all" (tudo). Executa todos os métodos de enumeração disponíveis:
- Listagem de usuários
- Grupos
- Shares (compartilhamentos de arquivos)
- Políticas de senha
- Informações do sistema
- 192.168.56.101 → IP do alvo (máquina vulnerável).
- | tee enum4_output.txt → redireciona a saída para o arquivo enum4_output.txt, mas também mostra os resultados na tela.
- tee é útil para salvar e visualizar ao mesmo tempo.

O enum4linux conecta ao host 192.168.56.101.
Executa múltiplas consultas via protocolos SMB/NetBIOS.
Coleta informações como:
Nome da máquina e domínio
Lista de usuários e grupos
Compartilhamentos disponíveis (ex.: C$, IPC$, ADMIN$)
Políticas de senha (complexidade, expiração)
Versão do sistema operacional
Tudo isso é salvo em enum4_output.txt para análise posterior.

# criação do arquivo texto com uma lista de nomes de usuários (um por linha: quebra de linha > \n)
echo -e '\user\nmsfadmin\nservice' > smb_users.txt

# criação do arquivo texto com uma lista de senhas (um por linha: quebra de linha > \n)
echo -e 'password\n123456\nWelcome123\nmsfadmin' > senhas_spray.txt

# ataque de força bruta contra o serviço SMB/NT (Server Message Block) da máquina alvo
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

- -h 192.168.56.101 → IP do host alvo.
- -U smb_users.txt → arquivo contendo lista de usuários SMB a serem testados.
- -P senhas_spray.txt → arquivo contendo lista de senhas a serem testadas.
- -M smbnt → módulo SMB/NTLM, usado para autenticação em serviços Windows/Samba.
- -t 2 → número de threads (2 tentativas simultâneas).
- -T 50 → número máximo de tentativas por thread antes de encerrar.

O Medusa conecta ao serviço SMB do host 192.168.56.101.
Para cada usuário listado em smb_users.txt, ele tenta autenticar usando as senhas de senhas_spray.txt.
Executa até 2 threads em paralelo, cada uma podendo realizar até 50 tentativas.
Se alguma combinação de usuário/senha for válida, o Medusa retorna o login encontrado.

# ferramenta smbclient para listar os compartilhamentos disponíveis em um servidor SMB (Windows/Samba).
smbclient -L //192.168.56.101 -U msfadmin

- smbclient → cliente SMB/CIFS usado para acessar compartilhamentos de arquivos em sistemas Windows ou Samba.
- -L → opção que significa listar os compartilhamentos disponíveis no servidor.
- //192.168.56.101 → endereço do servidor alvo (nesse caso, a máquina vulnerável).
- -U msfadmin → define o usuário que será usado para autenticação (nesse caso, msfadmin).

O comando tenta se conectar ao servidor SMB no IP 192.168.56.101.
Solicita autenticação com o usuário msfadmin (vai pedir a senha correspondente).
Se a autenticação for bem-sucedida, o smbclient lista os compartilhamentos disponíveis no servidor, como:
- IPC$ → canal de comunicação interno.
- ADMIN$ → diretório administrativo.
- C$ → raiz do disco C.
Outros compartilhamentos criados pelo administrador ou pelo sistema.
