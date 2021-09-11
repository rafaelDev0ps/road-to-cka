# Conhecimentos basicos de Redes
Os conceitos basicos serao divididos em topicos.  
- [Switching](#switching)
- [Routing](#routing)
- [DNS](#dns)
- [Dominios](#dominios)
  
## **Switching**  
Quando falamos de switching temos o objetivos de comunicar um host com outro em uma rede. Para isso precisamos configurar o IP de ambos os hosts na interface de rede usada. Nesse caso vamos supor que estamos usando a rede por meio fisico (cabo).  
Primeiro verificamos as interfaces disponiveis no host:  
```sh
$ ip link 
...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether dc:a6:32:94:2c:16 brd ff:ff:ff:ff:ff:ff
...
```  
Dentre elas temos a `eth0` responsavel pela interface com conexao fisica (cabo).  
Agora verificamos quais IPs estao anexados nas respectivas interfaces:  
```sh
$ ip addr show
...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether dc:a6:32:94:2c:16 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.21/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 61955sec preferred_lft 61955sec
    inet6 2001:1284:f028:799:dea6:32ff:fe94:2c16/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 86399sec preferred_lft 86399sec
    inet6 fe80::dea6:32ff:fe94:2c16/64 scope link 
       valid_lft forever preferred_lft forever
...
```  
Dentre varias opcoes encontramos o IP do nosso host `192.168.1.21/24` anexado a interface `eth0`.  
Isso significa que hosts que estiverem na mesma rede poderao se conectar com esse host atraves desse mesmo tipo de interface chamando por esse IP.  
  
Caso nao houvesse nenhum IP anexado a essa interface deveriamos adicionar esse IP nessa interface desejada:  
```sh
$ ip addr add 192.168.1.21 dev eth0
```  
A palavra `dev` significa `device` e define qual interface voce deseja adicionar esse endereco.  
  
**_Ok, mas e quando os hosts estao em redes diferentes? Como faco para conecta-los?_**  
Ai entra o roteamento...  
  
## Routing  
Quando queremos conectar hosts de redes diferentes precisamos usar um Router, para isso precisamos da nossa lista de IPs cadastrados no router para que a conexao funcione.  
```sh
$ route

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    100    0        0 eth0
default         _gateway        0.0.0.0         UG    600    0        0 wlan0
10.32.0.0       0.0.0.0         255.240.0.0     U     0      0        0 weave
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 wlan0
_gateway        0.0.0.0         255.255.255.255 UH    100    0        0 eth0
_gateway        0.0.0.0         255.255.255.255 UH    600    0        0 wlan0
```  
Acima temos a lista das rotas que o host conhece e eh capaz de fazer. Note que os Gateways com endereco `0.0.0.0` significa que nao usados, portanto os destinos estao na mesma rede.  
Alem disso temos o valor `_gateway` que possui o destino `default`. Ele representa o Gateway Default do host onde da acesso a Internet ou redes publicas.
  
Vamos supor que o outro host esta na rede `192.168.2.0/24`.  
  
Para criar uma rota para outro host fazemos:  
```sh
$ ip route add 192.168.2.0/24 via 192.168.1.1
```  
Dessa forma estamos vinculando a rede do outro host `192.168.2.0/24` com o endereco `192.168.1.1`.  
```sh
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.2.0     192.168.1.1     255.255.255.0   UG    100    0        0 eth0
```  
  
Vamos imaginar que temos 3 hosts A, B e C que estao representados abaixo.  
![Three Hosts](./assets/3_hosts.png)  
  
Para que A consiga se conectar com C eh necessario criar duas rotas.  
- Uma rota de A para B
- Uma rota de C para B
  
Dessa forma B podera encaminhar as chamadas de A para C e o contrario tambem. Portanto:  
```sh
$ ip route add 192.168.1.0 via 192.168.1.6  # executado no host A
$ ip route add 192.168.2.0 via 192.168.2.6  # executado no host C
```  
  
Ainda falta mais uma coisa, eh necessario habilitar o IP forwarding no host B. Por padrao no Linux essa funcionalidade eh desativada por motivos de seguranca. Assim sendo:  
```sh
$ cat /proc/sys/net/ipv4/ip_forward
0  # 0 => desabilitado
$ echo 1 > /proc/sys/net/ipv4/ip_forward
```  
Para que essa alteracao seja permanente precisamos configurar no arquivo `/etc/sysctl.conf`:  
```conf
net.ipv4.ip_forward = 1
```  
  
## DNS
Agora, se quisermos conectar de outra forma em um host? Existe outra maneira de conectar com um host sem que seja pelo endereco IPv4/IPv6 dele. Fazemos isso pelo DNS, podemos dizer de uma maneira mais didatica que o DNS eh um apelido para um endereco final.  
Podemos fazer isso configurando o arquivo `/etc/hosts`:  
```sh
$ echo 192.168.1.22    host2
```  
Assim podemos chamar de `host2` ao inves de `192.168.1.22`.  

Agora, imagine que exista inumeros host e cada host possui um nome (apelido). Quanto maior for essa lista de hosts configurados mais dificil sera dar manutencao. Para isso cria-se um servidor destinado para essa finalidade de resolver os nomes. Com isso, temos um Servidor DNS!  
  
Para que os hosts procurem por esse Servidor DNS precisamos configurar o arquivo `/etc/resolv.conf`:  
```conf
nameserver  192.168.1.76
```  
No exemplo acima o endereco `192.168.1.76` eh o IP do servidor DNS que ira resolver os nomes. Sendo assim, toda vez que o host fiz uma chamada para outro host ele ira chamar o servidor DNS para resolver o nome!  
  
### **Importante**
Mesmo que exista um servidor DNS definido o arquivo `/etc/hosts` sempre sera consultado antes de chamar o servidor DNS, portanto verifique se nao ha nomes duplicados ou com enderecos errados no Servidor DNS e no arquivo `/etc/hosts`.  
  
## Dominios
Para que seja possivel ter mais possibilidades de nomes foram criadas subdivisoes nos nomes e esses sao chamados de dominios. Vejamos um exemplo:  
> www.google.com  
Nesse caso o nome principal eh `.`. O primeiro dominio desse nome eh `.com` e o segundo eh `google`. Depois disso entram os subdominos, que nesse caso eh `www`.  
  
**_Como a busca desse nome eh feita?_**  
Quando buscamos por `www.google.com` acontece na seguinte ordem:  
1. A chamada vai para um servidor DNS raiz que resolve o nome `.`.
2. Depois eh redirecionado para outro servidor DNS responsavel por resolver dominios `.com`.
3. Em seguida eh redirecionado novamente para outro servidor DNS responsavel por resolver o nome `google` que tambem sera capaz de resolver os subdominions do dominio `google`, que no caso sera `www`.  
  
Para que as proximas buscas para esse dominio sejam mais rapidas o servidor DNS raiz armazena em cache o endereco do servidor DNS responsavel por resolver o dominio `google`.  
  
No nosso arquivo `/etc/resolve.conf` podemos configurar par que ele busque em um dominio ja conhecido, assim podemos informar somente o subdominio que pertence ao dominio configurado. Vejamos:  
```conf
search      meu-dominio.com
```  
No exemplo acima a palavra `search` ira indicar para que o host faca a busca do subdominio que desejo no servidor DNS responsavel pelo dominio `meu-dominio.com`. 
```sh
$ ping teste
PING teste.meu-dominio.com (192.168.3.24) 56(96) bytes of data.
```  
Veja que ele "autocompleta" o subdominio com o dominio configurado.  
  
Por ultimo, precisamos saber que existem varios tipos de registros de dominio. Segue a lista:  
- `A` representa o dominio que aponta para um endereco IPv4
- `AAAA` representa o dominio que aponta para um endereco IPv6
- `CNAME` representa os subdominios de um dominio  
  
Existem outros tipos mas esses daqui sao os mais usados.  
  
### **DICA**
Ferramentas para debugar um DNS:  
- `nslookup`
- `dig`
