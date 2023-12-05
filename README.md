## Configuração de Ambiente para coleta de dados de QoS (5QI) e Análise de dados utilizando Prometheus e NWDAF

O tutorial foi baseado no ambiente divulgado pelo Ciro Macedo no vídeo disponível em https://github.com/ciromacedo/UE-non3GPP-v1 e gravado em vídeo em https://youtu.be/UswQTnTZGt4

# Ambiente
Ambiente necessita de 4 máquinas para funcionar. O ambiente foi desenvolvido na AWS. Ao acessar deve-se criar 4 máquinas com os seguintes nomes e requisitos abaixo:
    - Máquina 1: free5GC
    - Máquina 2: N3IWF
    - Máquina 3: UE
    - Máquina 4: Máquina do Operador (openinstallcc)

SO: Ubuntu 20.04 (LTS) x64
Uname -r: 5.4.0-122-generic
Memory: 4 GB
Disk: 80 GB

Utilizado gerenciador de acesso remoto do VS Code para acessar as máquinas

# Permitir acesso a pasta AWS
Para liberar acesso ao gerenciamento de pastas caso utilizar um gerenciador de ssh deve permitir acesso ao usuário atual para editar as pastas, neste caso o usuário é ubuntu
```
sudo chown -R ubuntu /home/ubuntu/
```

# Instalar bibliotecas necessárias
```
sudo apt update && apt -y install python && sudo apt -y install git && sudo apt -y install ansible && sudo apt -y install net-tools
```

# Clone do projeto na máquina do operador (neste projeto openinstallcc)
```
apt update && git clone https://github.com/LABORA-INF-UFG/UE-non3GPP.git 
```

# Editar IP das máquinas no arquivo de conexão de hosts
Editar o arquivo em UE-non3GPP/dev/free5gc-v3.1.1/hosts e editar os IP's das máquinas de acesso e a interface de rede da máquina com o core através do comando abaixo no terminal linux e verificar qual nome da interface que faz conexão com a internet

```
ifconfig
```
A configuração do arquivo hosts deve ser feito da seguinte forma:

1: Colocar ip da máquina com o core (5gc)
2: Colocar a interface de conexão com a internet da máquina que hospeda o core
3: Colocar o ip da máquina com com a NF>n3iwf (n3iwf)
4: Colocar ip da máquina com o core (5gc)
5: Colocar ip da máquina com a UE (uecc)
6: Colocar o ip da máquina com com a NF>n3iwf (n3iwf)
7: Colocar ip da máquina com o core (5gc)

<p float="center">
    <img src="./img/hosts.png" width="800"/>
</p>
 O parâmetro "n3iwf_install=true" significa que deve subir o projeto Free5gc apenas com a NF n3iwf, caso seja "false" todas as NF serão habilitadas, exceto a n3iwf

# Habilitar o acesso ssh da máquina do operador para com as demais

Gerar chaves na máquina do operador:
```
ssh-keygen -t ecdsa -b 521
```

Permitir o acesso entre as máquinas, rodando o comando a seguir na máquina openinstallcc (pressionar enter 3x após rodar cada comando)
```
ssh-copy-id -i ~/.ssh/id_ecdsa.pub root@<free5gc-ip-address>
ssh-copy-id -i ~/.ssh/id_ecdsa.pub root@<n3iwf-ip-address>
ssh-copy-id -i ~/.ssh/id_ecdsa.pub root@<uecc-ip-address>
```
Caso o comando acima não funcione, pode ser utilizado o seguinte, para casos de servidor com acesso via chave .pem
```
cat ~/.ssh/id_ecdsa.pub | ssh -i "chave.pem" username@remote_host "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Teste de acesso de comunicação, lembre-se de liberar o icmp nas máquinas
```
sudo ansible -i ./hosts -m ping all -u ubuntu
```

# Install go
Antes de executar os comandos a seguir do tutorial deve-se verificar o usuário para acesso as máquinas, que por padrão dos scripts está com o root. Para tal deve-se verificar os arquivos:
1. UE-non3GPP/dev/free5gc-v3.1.1/free5gc-n3iwf-setup.yaml linha 4
2. UE-non3GPP/dev/free5gc-v3.1.1/go-install.yaml linha 4
3. UE-non3GPP/dev/UEnon3GPP-setup.yaml linha 4
Após alterar para o usuário de acesso as máquinas, podemos seguir para os demais passos para levantar o ambiente.

Instalando o Go.
```
ansible-playbook dev/free5gc-v3.1.1/go-install.yaml -i dev/free5gc-v3.1.1/hosts
```
Rodar o comando a seguir em cada um dos servers para atualizar as novas bibliotecas
```
source ~/.bashrc
```

# Setup Free5GC
Instalar curl para mongo individualmente em cada máquina
```
sudo apt-get install gnupg curl

curl -fsSL https://pgp.mongodb.com/server-6.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg \
   --dearmor
   
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

sudo apt-get update
```
Rodar comando seguinte na máquina openinstallcc
```
sudo ansible-playbook dev/free5gc-v3.1.1/free5gc-n3iwf-setup.yaml -i dev/free5gc-v3.1.1/hosts
```

# Setup UE-non3GPP with Ansible 
```
ansible-playbook dev/UEnon3GPP-setup.yaml -i dev/free5gc-v3.1.1/hosts
```
# Demo do ambiente
Com o ambiente online e funcional deve ser possível realizar teste de ping e acesso entre as máquinas, bem como gerar tráfego no túnel GTP, conforme a imagem abaixo.
<p float="center">
    <img src="./img/ping.png" width="800"/>
</p>

# Conclusão

Com isso todo o ambiente está pronto para iniciar, agora será necessário acessar os arquivos de configuração de IP de AMF, N3IWF entre outros que seguiremos mais adiante neste tutorial.