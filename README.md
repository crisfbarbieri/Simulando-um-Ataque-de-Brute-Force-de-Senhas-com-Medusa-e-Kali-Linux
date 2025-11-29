# Simulando-um-Ataque-de-Brute-Force-de-Senhas-com-Medusa-e-Kali-Linux
Desafio DIO - Implementar, documentar e compartilhar um projeto prático utilizando Kali Linux e a ferramenta Medusa, em conjunto com ambientes vulneráveis 

1. Configuração do Ambiente
Utilização do Kali Linux como sistema principal para execução dos testes.
Preparação de ambientes vulneráveis (máquina virtual metasploitable 2).

2. Ferramenta Utilizada: Medusa
Medusa foi escolhida por ser uma ferramenta de força bruta paralela voltada para testes de autenticação.
Configuração de parâmetros básicos: alvo, serviço, lista de usuários e senhas.

ping -c 192.168.56.101
Validado comunicação com a máquina vulnerável

nmap -sV -p 21,22,80,445,139 192.168.56.101 
Verifica se as portas listadas estão abertas, fechadas ou filtradas. Caso abertas, tenta identificar o serviço e versão rodando nelas


- nmap → Ferramenta de varredura de rede.
- -sV → Ativa a detecção de versão dos serviços. Ou seja, além de verificar se a porta está aberta, tenta identificar qual software e versão está rodando (ex.: Apache 2.4, OpenSSH 8.2).
- -p 21,22,80,445,139 → Define quais portas serão analisadas:
- 21 → FTP
- 22 → SSH
- 80 → HTTP (web)
- 445 → SMB (compartilhamento de arquivos no Windows)
- 139 → NetBIOS (antigo protocolo de compartilhamento no Windows)
- 192.168.56.101 → IP alvo dentro da rede local 

ftp 192.168.56.101
Tentativa de conexão com ftp

echo -e '\user\nmsfadmin\nadmin\nroot' > users.txt
criação do arquivo texto com uma lista de nomes de usuários (um por linha: quebra de linha > \n)

echo -e '123456\npassword\nqwerty\nmsfadmin' > pass.txt
criação do arquivo texto com uma lista de senhas (um por linha: quebra de linha > \n)

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6 
ataque de força bruta controlado usando a ferramenta Medusa contra o serviço FTP da máquina alvo .

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



