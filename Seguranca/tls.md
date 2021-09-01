# Certificados TLS
_Como funciona certificados TLS?_  
Esses certificados serverm para assegurar a encriptacao das requisicoes feitas de um host para outro. Ao inves de passar os dados como _plain-text_.  
  
Logo com o certificado, o cliente envia os dados encriptados com o certificado e quando o host receber esses dados ele ira conseguir le-los somente com a chave privada que desencripta os dados recebidos
  
### Tipos de encriptacao
- *Simetrica*: quando eh utilizado uma unica chave para encriptar os dados. Pode ser roubada e com isso pode acabar dando acesso aos dados pois a chave eh enviada junto com os dados para que o host consiga decriptar.
- *Assimetrica*: existem duas chaves, uma publica e uma privada. A chave publica serve para encriptar os dados no cliente, enquanto a privada fica com o host para conseguir decriptar os dados recebidos.
  
_Como gerar um par de chaves?_
```sh
ssh-keygen
```  
Com isso a chave publica fica para o cliente enquato a  chave privada e salva com o host.  
Podemos fazer isso tambem com a ferramenta `openssl`.  
```sh
openssl genrsa -out minha-chave.key 1024
openssl rsa -in minha-chave.key -pubout > minha-chave-privada.pem
```  
  
Para melhorar ainda mais a seguranca, eh utilizado um Certificado de Autoridade (CA). Uma CA eh autenticada pelo navegador quando uma requisicao eh feita atraves dele, onde existe uma sessao de CAs confiaveis e com isso eh validado. Quando uma requisicao eh feita internalmente eh necessario utilizar uma CA privada sendo validada por um servidor responsavel por essas validacoes. O servidor pode ser hosteado tanto na sua infra quanto em um endpoint de terceiros.  
  
**Esse metodo que utiliza certificados e chaves privadas eh chamado de PKI Public Key Infrastructure**  
  
Seguindo uma convencao, chaves publicas devem ser usadas com as extencoes `.crt` ou `.pem`. Enquanto chaves privadas devem ser usadas com `.key` ou `-key.pem`.