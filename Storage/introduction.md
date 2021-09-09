# Introducao ao Storage
Para entendermos melhor como funciona o storage no K8s precisamos entender como o Docker gerencia seus arquivos.  
  
## Docker filesystem
Quando escrevemos uma image no Docker cada comando representa uma camada. Um container eh composto por varias camadas. Veja o exemplo a seguir:  
```dockerfile
FROM ubuntu:20.04
COPY . /app
RUN make /app
CMD python /app/app.py
```  
Nesse exemplo o container possui 4 comandos, sendo o primeiro a camada com a imagem base `ubuntu:20:04`, a segunda camada faz a copia dos arquivos do diretorio `/app`. A terceira executa o comando `make` e a quarta executa o programa escrito em Python com o comando `python`.  

Porem nao existe somente 4 camadas nesse cenario, ainda ha uma ultima camada chamada _Container Layer_, ou Camada do Container.  

## Container Layer
Nessa camada fica localizado todos os arquivos temporarios do container como logs da aplicacao, arquivos temporarios que serao manipulados, ou arquivos adicionados pelo usuario.  
Esses arquivos sao guardados no diretorio `/var/lib/docker` que eh o diretorio padrao onde o docker salva dos dados de cada camada criada.  

**Observacao:** os arquivos na _Container Layer_ possuem permissao de leituira e escrita enquanto nas camadas abaixo dela somente possuem permissao de leitura.  
  
### _Ok, mas os arquivos do meu projeto nao estao na Container Layer, se eu quiser altera-los como eu faco?_  
Nesse caso o Docker acaba fazendo uma copia desse arquivo e adiciona na _Container Layer_. Esse processo se chama COPY-ON-WRITE.  
  
## Como persistir dados em um container?
Nesse caso eh necessario fazer um _bind_ de um volume com o seguinte comando:
```sh
docker run -v data_volume:/var/lib/mysql mysql
```  
Com isso os dados serao salvos no arquivo padrao onde o Docker faz o _mount_ dos volumes em `/var/lib/docker/volumes/data_volume`. Isso eh chamado de _Volume Mounting_, quando criamos um diretorio do zero dentro do conatainer.  
  
Agora caso queiramos anexar dados em outro local precisamos passar o caminho completo, veja abaixo:  
```sh
docker run -v /data/meu-app:/usr/app/meu-app meu-app
```  
Isso se chama _Bind Mounting_, quando pegamos um diretorio ja existente e montamos dentro do container.  
  
## Storage Drivers
Para fazer tudo isso o Docker utiliza Storage Drivers, eles podem variar dependendo de cada SO onde o container esta sendo executado.  
Existem os seguintes Storage Drivers:  
- AUFS
- ZFS
- BTRFS
- Device Mapper
- Overlay
- Overlay2