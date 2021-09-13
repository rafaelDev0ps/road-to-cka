# Docker Network
Para entender melhor como funciona Redes dentro do cluster K8s eh necessario entender como funciona Redes em containeres.  
  
Existem 3 tipos de rede no Docker.  
```sh
$ docker network ls

NETWORK ID     NAME      DRIVER    SCOPE
6e6ceef07912   bridge    bridge    local
21af5f9db46f   host      host      local
649b311a4f20   none      null      local
```
  
### Network NONE
Quando criamos um container atraves do Docker o proprio container nao consegue acessar hosts externos nem a internet. O contrario tambem eh verdadeiro, ninguem de fora consegue acessar o container atraves de uma rede externa.  
Por padrao quando o container eh criado as configuracoes de rede vem da seguinte maneira:  
```sh
docker run --network none ubuntu
```  
Veja que o valor do argumento `--network` eh `none`, ou seja, nao ah uma rede para esse container.  
  
Sendo assim, para adicionar uma rede para que ele seja acessivel existe a primeira forma que eh definir uma rede do tipo `host`.  
  
### Network HOST
Uma rede do tipo host utiliza a mesma interface de rede da maquina local (seja de uma instancia ou do seu proprio PC). Vejamos:  
```sh
docker run --network host meu-app-frontend
```  
Com isso se voce definir que seu app `meu-app-frontend` rode na porta 8090 do container, ele ira se apropriar da porta 8090 da maquina local. Vamo supor que voce esteja rodando esse container no seu computador pessoal e que o endereco da interface de rede seja `192.168.1.10`.  Logo essa app podera ser acessada em `http://192.168.1.10:8090`.  
  
E existe outro tipo de rede que eh o tipo `bridge`...  
  
### Network BRIDGE
Esse tipo de rede eh mais interessante que o `host`. Esse tipo eh capaz de criar uma interface de rede propria do Docker que podemos ver atraves do comando `ip link`.  
```sh
$ ip link
...
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:a3:03:7e:f8 brd ff:ff:ff:ff:ff:ff
...
```  
Conseguimos ver qual endereco esta anexado a essa interface atraves do comando `ip addr`.  
```sh
$ ip addr

4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:a3:03:7e:f8 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a3ff:fe03:7ef8/64 scope link 
       valid_lft forever preferred_lft forever
```  
Ele utiliza o endereco `172.17.0.1/16`!  
  
Quando um container eh criado atraves do Docker ele automaticamente adiciona um Namespace de Rede proprio para o container (caso ainda nao tenha visto sobre Network Namespaces veja [aqui](https://github.com/rafaelDev0ps/road-to-cka/blob/main/Networking/basics.md#network-namespaces)).  
  
Se inspecionarmos o container veremos o namespace criado para o container.  
```sh
$ docker inspect meu-container

...
"NetworkSettings": {
  "Bridge": "",
  "SandboxID": "46a8923ffa1ed04ce37c273f6cce4c038462a90f176277677d470e3ab07c4750",
  "HairpinMode": false,
  "LinkLocalIPv6Address": "",
  "LinkLocalIPv6PrefixLen": 0,
  "Ports": {},
  "SandboxKey": "/var/run/docker/netns/46a8923ffa1e",  # <== AQUI O NAMESPACE 46a8923ffa1e
  "SecondaryIPAddresses": null,
...
```  
  
Para acessar uma app que usar rede do tipo `bridge` eh necessario especificar uma segunda porta para que o Docker possa fazer o port-forward da porta do container.  
```sh
$ docker run -p 8080:8090 meu-app-frontend
```  
Com isso o voce ira conseguir acessar seu app atraves da porta `8080` em `http://172.17.0.1:8080` e as chamadas serao redirecionadas para a porta do container que eh `8090`.  
  
Esse port-forward acontece pois ele utiliza um `iptables` e realiza um NAT de uma porta para outra. Algo semelhante a isso aqui:  
```sh
$ iptables -t NAT -A PREROUTING -j DNAT --dport 8080 --to-destination 8090
```  
  
Voce pode listar todas essas regras feitas pelo Docker atraves do iptables.  
```sh
$ iptables -nvL -t nat
```