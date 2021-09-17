# Falhas nas aplicacaoes
O que fazer quando as aplciacoes instaladas no cluster falham?
  
Services
- Checar eventos (describe)
- Conferir portas
- Conferir selectors
- Conferir tipo de service
- Checar se o endpoint corresponde ao IP do Pod de destino
- Conferir namespace

Deployments
- Checar eventos (describe)
- Conferir env var
- Conferir labels
- Checar logs do Pod do deploy
  
Pods
- Checar eventos (describe)
- Conferir labels
- Checar logs
- Checar volumes montados
- Executar um pod internamente
- Tentar acessar outro Pod dentro de um Pod
- Verificar recursos