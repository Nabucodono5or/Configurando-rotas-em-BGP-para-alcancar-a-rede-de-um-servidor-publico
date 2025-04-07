# Configurando-rotas-em-BGP-para-alcancar-a-rede-de-um-servidor-publico

## **Descrição**
Em um ambiente onde temos diversos AS (autnomous system), para alcançar um servidor, fora configurado maneiras disso ocorrer de um operadora local, ao mesmo tempo em que fora configurado também outras rotas divulgadas por AS específicos , além de controle pqra que a operadora não se tronasse um As transitório para rotas externas.

Projeto baseado na topologia construída pelo Youtuber Matheus Leal no vídeo **#17 - CONFIGURE BGP NO MODO EASY**

## **Topologia**

A topologia seguida para o projeto.

![topologia](Pasted%20image%2020250407105621.png)

O meu foco aqui seria como o a operadora, router **OPERADORA-LOCAL** de AS 10, lidaria com as redes aprendidas externamente do AS 65001, e também da rede **FACEBOOK**, que simulam endereços para os seus servidores de serviços.

Os outros AS são trânsitos para essas rotas e estarão na luta para ser candidato a melhor rotas.

Temos as seguintes configurações da INTERNET AS 65001, e CLIENTES. Ambos aqui são servidores linux gerando rotas.

### CLIENTES

![topologia](Pasted%20image%2020250407110228.png)

### INTERNET

![topologia](Pasted%20image%2020250407110318.png)

Como se pode notar com RID 1.1.1.1 a rede CLIENTES divulga rotas ospf para ao nó OPERADORA-LOCAL. A ideia é que estamos compartilhando rotas de uma rede local a operadora.

Configuração dos protocolos em cada nó.

### OPERADORA-LOCAL

```julia
OPERADORA#sh running-config | section bgp
router bgp 10
 no synchronization
 bgp log-neighbor-changes
 redistribute ospf 1 route-map EXP-REDE-LOCAL
 neighbor 199.10.40.2 remote-as 400
 neighbor 199.10.40.2 route-map REDE-ISP400-IN in
 neighbor 199.10.40.2 route-map EXP-REDE-LOCAL out
 neighbor 199.10.100.2 remote-as 100
 neighbor 199.10.100.2 route-map BLC-REDE-IN in
 neighbor 199.10.100.2 route-map EXP-REDE-LOCAL out
 no auto-summary
 !
OPERADORA#sh running-config | section ospf
 ip ospf 1 area 0
router ospf 1
 router-id 10.10.10.10
 log-adjacency-changes
 redistribute ospf 1 route-map EXP-REDE-LOCAL

```

As route-map serão explicadas melhor, ao longo do documento.

### ISP400

```julia
ISP400#sh running-config | section bgp
router bgp 400
 no synchronization
 bgp log-neighbor-changes
 neighbor 199.10.40.1 remote-as 10
 neighbor 199.40.40.1 remote-as 500
 no auto-summary
```

### ISP100

```julia
ISP100#show running-config | section bgp
router bgp 100
 no synchronization
 bgp log-neighbor-changes
 neighbor 199.10.100.1 remote-as 10
 neighbor 199.100.50.1 remote-as 500
 neighbor 199.100.200.2 remote-as 200
 no auto-summary

```

### ISP200

```julia
ISP200#show running-config | section bgp
router bgp 200
 no synchronization
 bgp log-neighbor-changes
 neighbor 199.30.200.1 remote-as 300
 neighbor 199.100.200.1 remote-as 100
 neighbor 200.14.1.2 remote-as 65001
 no auto-summary
```

### ISP500

```julia
ISP500#sh running-config | section bgp
router bgp 500
 no synchronization
 bgp log-neighbor-changes
 neighbor 199.40.40.2 remote-as 400
 neighbor 199.50.99.1 remote-as 1000
 neighbor 199.100.50.2 remote-as 100
 no auto-summary

```

### ISP300

```julia
ISP300#show running-config | section bgp
router bgp 300
 no synchronization
 bgp log-neighbor-changes
 neighbor 199.30.99.1 remote-as 1000
 neighbor 199.30.200.2 remote-as 200
 no auto-summary

```

### FACEBOOK

```julia
facebook#sh running-config | section bgp
router bgp 1000
 no synchronization
 bgp log-neighbor-changes
 network 31.13.64.64 mask 255.255.255.255
 network 31.13.73.73 mask 255.255.255.255
 network 31.13.74.74 mask 255.255.255.255
 neighbor 199.30.99.2 remote-as 300
 neighbor 199.30.99.2 route-map BLC-REDE-IN in
 neighbor 199.30.99.2 route-map BLC-REDE-OUT out
 neighbor 199.50.99.2 remote-as 500
 neighbor 199.50.99.2 route-map BLC-REDE-IN in
 neighbor 199.50.99.2 route-map BLC-REDE-OUT out
 no auto-summary
```

O FACEBOOK aqui é um router simulando configuração de redes.

## **Endereçamento**

Endereçamentos IPv4 para o cenário utilizado.

| DISPOSITIVO     | INTERFACE  | ENDEREÇO IPV4 |
| --------------- | ---------- | ------------- |
| OPERADORA-LOCAL | F0/0       | 199.10.100.1  |
| OPERADORA-LOCAL | F0/1       | 199.10.40.1   |
| OPERADORA-LOCAL | F1/0       | 100.100.100.1 |
| ISP100          | F0/0       | 199.10.100.2  |
| ISP100          | F0/1       | 199.100.200.1 |
| ISP100          | F1/0       | 199.100.50.2  |
| ISP200          | F0/0       | 200.14.1.1    |
| ISP200          | F0/1       | 199.100.200.2 |
| ISP200          | F1/0       | 199.30.200.2  |
| ISP300          | F0/0       | 199.30.99.2   |
| ISP300          | F1/0       | 199.30.200.1  |
| ISP400          | F0/0       | 199.40.40.2   |
| ISP400          | F0/1       | 199.10.40.2   |
| ISP500          | F0/0       | 199.40.40.1   |
| ISP500          | F0/1       | 199.50.99.2   |
| ISP500          | F1/0       | 199.100.50.1  |
| FACEBOOK        | F0/0       | 199.30.99.1   |
| FACEBOOK        | F0/1       | 199.50.99.1   |
| FACEBOOK        | LOOPBACK64 | 31.13.64.64   |
| FACEBOOK        | LOOPBACK73 | 31.13.73.73   |
| FACEBOOK        | LOOPBACK74 | 31.13.74.74   |

## **Configuração**

As seguintes configurações foram feitas nos nós presentes.

### OPERADORA-LOCAL

```julia
conf t
router ospf 1
router-id 10.10.10.10
exit
interface FastEthernet 1/0
ip ospf 1 area 0
exit

ip prefix-list REDE-LOCAL seq 10 permit 100.100.0.0/24
ip prefix-list REDE-LOCAL seq 20 permit 100.100.0.0/24 ge 25
ip prefix-list REDE-LOCAL seq 30 permit 100.100.1.0/24
ip prefix-list REDE-LOCAL seq 40 permit 100.100.1.0/24 ge 25

route-map BLC-REDE-IN deny 10
match ip address prefix-list REDE-LOCAL
exit

route-map BLC-REDE-IN permit 20
exit

route-map EXP-REDE-LOCAL permit 10
match ip address prefix-list REDE-LOCAL
exit

router bgp 10
redistribute ospf 1 route-map EXP-REDE-LOCAL
neighbor 199.10.100.2 remote-as 100
neighbor 199.10.40.2 remote-as 400
neighbor 199.10.100.2 route-map BLC-REDE-IN in
neighbor 199.10.100.2 route-map EXP-REDE-LOCAL out
neighbor 199.10.40.2 route-map BLC-REDE-IN in
neighbor 199.10.40.2 route-map EXP-REDE-LOCAL out
end

ip prefix-list REDE-EXTERNA seq 10 permit 31.13.0.0/16 le 32
!
route-map REDE-ISP400-IN permit 10
match ip address prefix-list REDE-EXTERNA
set local-preference 300
exit
!
route-map REDE-ISP400-IN permit 20
exit
!
router bgp 10
neighbor 199.10.40.2 route-map REDE-ISP400-IN in
end

```

Onde configuramos para que somente endereços de divulgação para a rede da operadora fossem divulgados. E apesar de não ter impacto direto a comunicação da rede FACEBOOK, foi configurado local preference para o AS 400, sendo que tanto pelo AS 100 o caminho preferido, seria a interface F0/1 da rede FACEBOOK, contudo para controle se optou cerfitifcar qual seria a rota principal para impedir problemas de sincronização.  

### ISP400

```julia
! ISP 400
conf t
router bgp 400
neighbor 199.40.40.1 remote-as 500
neighbor 199.10.40.1 remote-as 10
end
```

### ISP100

```julia
! ISP 100
conf t
router bgp 100
neighbor 199.100.50.1 remote-as 100
neighbor 199.100.200.2 remote-as 200
neighbor 199.10.100.2 remote-as 10
end
```

### ISP200

```julia
! ISP 200
conf t
router bgp 200
neighbor 199.100.200.1 remote-as 100
neighbor 199.30.200.1 remote-as 300
neighbor 200.14.1.2 remote-as 65001
end
```

### ISP500

```julia
! ISP500
conf t
router bgp 500
neighbor 199.50.99.1 remote-as 1000
neighbor 199.100.50.2 remote-as 100
neighbor 199.40.40.2 remote-as 400
end
```

### ISP300

```julia
! ISP300
conf t
router bgp 300
neighbor 199.30.99.1 remote-as 1000
neighbor 199.30.200.2 remote-as 200
! neighbor 199.30.99.2 remote-as 400
end
```

### FACEBOOK

```julia
! facebook
conf t
router bgp 1000
network  31.13.64.64 mask 255.255.255.255
network  31.13.73.73 mask 255.255.255.255
network  31.13.74.74 mask 255.255.255.255
neighbor 199.50.99.2 remote-as 500
neighbor 199.30.99.2 remote-as 300
exit
!
ip prefix-list LOCAL-REDE seq 10 permit 31.13.64.64/32
ip prefix-list LOCAL-REDE seq 20 permit 31.13.64.64/15 ge 16
!
route-map BLC-REDE-IN deny 10
match ip address prefix-list LOCAL-REDE
exit
!
route-map BLC-REDE-IN permit 20
exit
!
route-map BLC-REDE-OUT permit 10
match ip address prefix-list LOCAL-REDE
exit
!
router bgp 1000
neighbor 199.50.99.2 route-map BLC-REDE-IN in
neighbor 199.30.99.2 route-map BLC-REDE-IN in
neighbor 199.50.99.2 route-map BLC-REDE-OUT out
neighbor 199.30.99.2 route-map BLC-REDE-OUT out
end
```

No facebook tempos filtro para que essa rede também não virasse transitória para outras rotas. E também fora configurado os endereços dos servidores em interfaces de loopback.

## **Testes e Validação**

Validando rotas, route-maps e prefix-lists da operadora para o destino FACEBOOK.

### OPERADORA-LOCAL

```julia
OPERADORA#show ip bgp 
BGP table version is 50, local router ID is 199.10.100.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  2.2.2.2/32       199.10.40.2                            0 400 500 100 200 65001 i
*>                  199.10.100.2                           0 100 200 65001 i
*  31.13.64.64/32   199.10.100.2                           0 100 500 1000 i
*>                  199.10.40.2                   300      0 400 500 1000 i
*  31.13.73.73/32   199.10.100.2                           0 100 500 1000 i
*>                  199.10.40.2                   300      0 400 500 1000 i
*  31.13.74.74/32   199.10.100.2                           0 100 500 1000 i
*>                  199.10.40.2                   300      0 400 500 1000 i
*  99.0.0.0/32      199.10.40.2                            0 400 500 100 200 65001 ?
*>                  199.10.100.2                           0 100 200 65001 ?
*  99.0.0.1/32      199.10.40.2                            0 400 500 100 200 65001 ?
*>                  199.10.100.2                           0 100 200 65001 ?
*  99.0.0.2/32      199.10.40.2                            0 400 500 100 200 65001 ?
   Network          Next Hop            Metric LocPrf Weight Path
*>                  199.10.100.2                           0 100 200 65001 ?
*  99.0.0.3/32      199.10.40.2                            0 400 500 100 200 65001 ?
*>                  199.10.100.2                           0 100 200 65001 ?
*  99.0.0.4/32      199.10.40.2                            0 400 500 100 200 65001 ?
*>                  199.10.100.2                           0 100 200 65001 ?
*  99.0.0.5/32      199.10.40.2                            0 400 500 100 200 65001 ?
*>                  199.10.100.2                           0 100 200 65001 ?
*  99.0.0.6/32      199.10.40.2                            0 400 500 100 200 65001 ?
*>                  199.10.100.2                           0 100 200 65001 ?
*  99.0.0.7/32      199.10.40.2                            0 400 500 100 200 65001 ?
*>                  199.10.100.2                           0 100 200 65001 ?
*  99.0.0.8/32      199.10.40.2                            0 400 500 100 200 65001 ?
*>                  199.10.100.2                           0 100 200 65001 ?
*  99.0.0.9/32      199.10.40.2                            0 400 500 100 200 65001 ?
*>                  199.10.100.2                           0 100 200 65001 ?
   Network          Next Hop            Metric LocPrf Weight Path
*> 100.100.0.1/32   100.100.100.2           31         32768 ?
*> 100.100.0.2/32   100.100.100.2           31         32768 ?
*> 100.100.0.3/32   100.100.100.2           31         32768 ?
*> 100.100.0.4/32   100.100.100.2           31         32768 ?
*> 100.100.0.5/32   100.100.100.2           31         32768 ?
*> 100.100.0.6/32   100.100.100.2           31         32768 ?
*> 100.100.0.7/32   100.100.100.2           31         32768 ?
*> 100.100.0.8/32   100.100.100.2           31         32768 ?
*> 100.100.0.9/32   100.100.100.2           31         32768 ?
*> 100.100.0.10/32  100.100.100.2           31         32768 ?

OPERADORA#sho route-map 
route-map EXP-REDE-LOCAL, permit, sequence 10
  Match clauses:
    ip address prefix-lists: REDE-LOCAL 
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
route-map BLC-REDE-IN, deny, sequence 10
  Match clauses:
    ip address prefix-lists: REDE-LOCAL 
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
route-map BLC-REDE-IN, permit, sequence 20
  Match clauses:
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
route-map REDE-ISP400-IN, permit, sequence 10
  Match clauses:
    ip address prefix-lists: REDE-EXTERNA 
  Set clauses:
    local-preference 300
  Policy routing matches: 0 packets, 0 bytes
route-map REDE-ISP400-IN, permit, sequence 20
  Match clauses:
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes

OPERADORA#show ip prefix-list REDE-LOCAL
ip prefix-list REDE-LOCAL: 4 entries
   seq 10 permit 100.100.0.0/24
   seq 20 permit 100.100.0.0/24 ge 25
   seq 30 permit 100.100.1.0/24
   seq 40 permit 100.100.1.0/24 ge 25

OPERADORA#show ip prefix-list REDE-EXTERNA
ip prefix-list REDE-EXTERNA: 1 entries
   seq 10 permit 31.13.0.0/16 le 32

OPERADORA#show ip protocols 
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 10.10.10.10
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
  Routing on Interfaces Configured Explicitly (Area 0):
    FastEthernet1/0
 Reference bandwidth unit is 100 mbps
  Routing Information Sources:
    Gateway         Distance      Last Update
    2.2.2.2              110      03:19:05
  Distance: (default is 110)

Routing Protocol is "bgp 10"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  IGP synchronization is disabled
  Automatic route summarization is disabled
  Redistributing: ospf 1
  Neighbor(s):
    Address          FiltIn FiltOut DistIn DistOut Weight RouteMap
    Address          FiltIn FiltOut DistIn DistOut Weight RouteMap
    199.10.40.2                                           REDE-ISP400-IN
    199.10.100.2                                          BLC-REDE-IN
  Maximum path: 1
  Routing Information Sources:
    Gateway         Distance      Last Update
    (this router)        200      01:38:54
    199.10.100.2          20      03:19:41
    199.10.40.2           20      03:20:33
  Distance: external 20 internal 200 local 200

OPERADORA#sho ip route | begin Gateway
Gateway of last resort is not set

     199.10.40.0/30 is subnetted, 1 subnets
C       199.10.40.0 is directly connected, FastEthernet0/1
     2.0.0.0/32 is subnetted, 1 subnets
B       2.2.2.2 [20/0] via 199.10.100.2, 03:20:29
     100.0.0.0/8 is variably subnetted, 11 subnets, 2 masks
O E1    100.100.0.1/32 [110/31] via 100.100.100.2, 03:19:55, FastEthernet1/0
O E1    100.100.0.2/32 [110/31] via 100.100.100.2, 03:19:55, FastEthernet1/0
O E1    100.100.0.3/32 [110/31] via 100.100.100.2, 03:19:55, FastEthernet1/0
O E1    100.100.0.4/32 [110/31] via 100.100.100.2, 03:19:55, FastEthernet1/0
O E1    100.100.0.5/32 [110/31] via 100.100.100.2, 03:19:55, FastEthernet1/0
O E1    100.100.0.6/32 [110/31] via 100.100.100.2, 03:19:55, FastEthernet1/0
O E1    100.100.0.7/32 [110/31] via 100.100.100.2, 03:19:55, FastEthernet1/0
O E1    100.100.0.8/32 [110/31] via 100.100.100.2, 03:19:55, FastEthernet1/0
O E1    100.100.0.9/32 [110/31] via 100.100.100.2, 03:19:55, FastEthernet1/0
O E1    100.100.0.10/32 [110/31] via 100.100.100.2, 03:19:56, FastEthernet1/0
C       100.100.100.0/30 is directly connected, FastEthernet1/0
     99.0.0.0/32 is subnetted, 10 subnets
B       99.0.0.3 [20/0] via 199.10.100.2, 03:20:30
B       99.0.0.2 [20/0] via 199.10.100.2, 03:20:30
B       99.0.0.1 [20/0] via 199.10.100.2, 03:20:30
B       99.0.0.0 [20/0] via 199.10.100.2, 03:20:30
B       99.0.0.7 [20/0] via 199.10.100.2, 03:20:32
B       99.0.0.6 [20/0] via 199.10.100.2, 03:20:32
B       99.0.0.5 [20/0] via 199.10.100.2, 03:20:32
B       99.0.0.4 [20/0] via 199.10.100.2, 03:20:32
B       99.0.0.9 [20/0] via 199.10.100.2, 03:20:32
B       99.0.0.8 [20/0] via 199.10.100.2, 03:20:32
     199.10.100.0/30 is subnetted, 1 subnets
C       199.10.100.0 is directly connected, FastEthernet0/0
     31.0.0.0/32 is subnetted, 3 subnets
B       31.13.74.74 [20/0] via 199.10.40.2, 03:21:23
B       31.13.73.73 [20/0] via 199.10.40.2, 03:21:23
B       31.13.64.64 [20/0] via 199.10.40.2, 03:21:23

```


### FACEBOOK

```julia
facebook#show ip bgp 
BGP table version is 55, local router ID is 31.13.74.74
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  2.2.2.2/32       199.50.99.2                            0 500 100 200 65001 i
*>                  199.30.99.2                            0 300 200 65001 i
*> 31.13.64.64/32   0.0.0.0                  0         32768 i
*> 31.13.73.73/32   0.0.0.0                  0         32768 i
*> 31.13.74.74/32   0.0.0.0                  0         32768 i
*  99.0.0.0/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
*  99.0.0.1/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
*  99.0.0.2/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
*  99.0.0.3/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
*  99.0.0.4/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
*  99.0.0.5/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
   Network          Next Hop            Metric LocPrf Weight Path
*  99.0.0.6/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
*  99.0.0.7/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
*  99.0.0.8/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
*  99.0.0.9/32      199.50.99.2                            0 500 100 200 65001 ?
*>                  199.30.99.2                            0 300 200 65001 ?
*  100.100.0.1/32   199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?
*  100.100.0.2/32   199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?
*  100.100.0.3/32   199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?
*  100.100.0.4/32   199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?
*  100.100.0.5/32   199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?
*  100.100.0.6/32   199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?
*  100.100.0.7/32   199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?
   Network          Next Hop            Metric LocPrf Weight Path
*  100.100.0.8/32   199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?
*  100.100.0.9/32   199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?
*  100.100.0.10/32  199.30.99.2                            0 300 200 100 10 ?
*>                  199.50.99.2                            0 500 400 10 ?

facebook#show route-map 
route-map BLC-REDE-OUT, permit, sequence 10
  Match clauses:
    ip address prefix-lists: LOCAL-REDE 
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
route-map BLC-REDE-IN, deny, sequence 10
  Match clauses:
    ip address prefix-lists: LOCAL-REDE 
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes
route-map BLC-REDE-IN, permit, sequence 20
  Match clauses:
  Set clauses:
  Policy routing matches: 0 packets, 0 bytes

facebook#show ip prefix-list LOCAL-REDE
ip prefix-list LOCAL-REDE: 2 entries
   seq 10 permit 31.13.64.64/32
   seq 20 permit 31.12.0.0/15 ge 16

facebook#show ip route | begin Gateway 
Gateway of last resort is not set

     2.0.0.0/32 is subnetted, 1 subnets
B       2.2.2.2 [20/0] via 199.30.99.2, 03:22:56
     100.0.0.0/32 is subnetted, 10 subnets
B       100.100.0.1 [20/0] via 199.50.99.2, 01:39:15
B       100.100.0.2 [20/0] via 199.50.99.2, 01:39:15
B       100.100.0.3 [20/0] via 199.50.99.2, 01:39:15
B       100.100.0.4 [20/0] via 199.50.99.2, 01:39:15
B       100.100.0.5 [20/0] via 199.50.99.2, 01:39:15
B       100.100.0.6 [20/0] via 199.50.99.2, 01:39:15
B       100.100.0.7 [20/0] via 199.50.99.2, 01:39:15
B       100.100.0.8 [20/0] via 199.50.99.2, 01:39:15
B       100.100.0.9 [20/0] via 199.50.99.2, 01:39:15
B       100.100.0.10 [20/0] via 199.50.99.2, 01:39:15
     99.0.0.0/32 is subnetted, 10 subnets
B       99.0.0.3 [20/0] via 199.30.99.2, 03:22:56
B       99.0.0.2 [20/0] via 199.30.99.2, 03:22:56
B       99.0.0.1 [20/0] via 199.30.99.2, 03:22:56
B       99.0.0.0 [20/0] via 199.30.99.2, 03:22:56
B       99.0.0.7 [20/0] via 199.30.99.2, 03:22:56
B       99.0.0.6 [20/0] via 199.30.99.2, 03:22:58
B       99.0.0.5 [20/0] via 199.30.99.2, 03:22:58
B       99.0.0.4 [20/0] via 199.30.99.2, 03:22:58
B       99.0.0.9 [20/0] via 199.30.99.2, 03:22:58
B       99.0.0.8 [20/0] via 199.30.99.2, 03:22:58
     199.30.99.0/30 is subnetted, 1 subnets
C       199.30.99.0 is directly connected, FastEthernet0/0
     199.50.99.0/30 is subnetted, 1 subnets
C       199.50.99.0 is directly connected, FastEthernet0/1
     31.0.0.0/32 is subnetted, 3 subnets
C       31.13.74.74 is directly connected, Loopback74
C       31.13.73.73 is directly connected, Loopback73
C       31.13.64.64 is directly connected, Loopback64

```
