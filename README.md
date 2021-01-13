# script-dynamicIp-wsl2
Script em powershell para setar ip dinânico ao wsl2 usando ip local do windows.

1. Abra o Powershell como administrador;

2. Execute o comando ```Set-ExecutionPolicy unrestricted```  em seguida execute ```Get-ExecutionPolicy;```, Caso apareça "Unrestricted" o comando funcionou.

3. Abra um bloco de notas e adicione o seguinte script:
```
$remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if( $found ){
  $remoteport = $matches[0];
} else{
  echo "O script não funcionou  pois endereço IP do WSL 2 não pode ser encontrado";
  exit;
}

#[Portas]

#Todas as portas que você deseja encaminhar separadas por vírgulas
$ports=@(80,443,10000,3000,5000,8000,8001);


#[IP estatico]
#Você pode alterar o endereço de configuração do seu ip para para um endereço específico PS: no meu caso estou usando ip local da minha maquina(windows)
#Caso não saiba o ip local, execute o comando "ipconfig" no powershell
$addr='000.000.000.000';
$ports_a = $ports -join ",";


#Remove as regras de exceção de firewall
iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";

#Adiciona as regras de exceção para as regras de entrada e saída
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";

for( $i = 0; $i -lt $ports.length; $i++ ){
  $port = $ports[$i];
  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport";
}
```
Salve o arquivo em extensão .ps1 (script powershell), exemplo. "script.ps1".

4. Agora vamos agendar a execução do script, para isso abra o executar win+r e escreva taskschd.msc. Após abrir o Agendador de tarefas clique em 
"Criar tarefa..." que se encontra no canto direito da tela.
5. Na aba **Geral** dê o nome e descrição a tarefa e selecione o checkbox **✅ Executar com privilégios mais altos.**

6. Na aba **Disparadores** clique em "novo...", após isso configure da seguinte forma. Iniciar tarega : "Ao fazer logon", Selecione um usuário em especifico caso queira , nas configurações avançadas marque: **✅ Atrasar a tarefa em**, digite o valor de "10 segundos" e clique em "Ok".

7. Na aba **Ações** clique em "novo...", após abrir a tela em "Ação" selecione a opção "Iniciar um programa", nas Configurações  Programa/script escreve "powershell" e em "Adicionar argumentos" escreva -ExecutionPolicy Bypass -File "C:\local do script\script.ps1", edite para o caminho do seu script após clique em "ok".

8. EXTRA, caso queira definir um nome "dominio" para o ip basta editar como administrador  o arquivo "C:\Windows\System32\drivers\etc" (Eu uso notepad++ que facilita a execução como administrador) e acrescentar a linha "SEUIPLOCAL  NOMEDODOMINIO" no meu caso ficou "192.168.1.5			dev.song".

Basta reiniciar o computador para testar, espero ter ajudado e até...😃😃😃 
