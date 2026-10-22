# Laboratório: NAT Estático e NAT Dinâmico

## Objetivo

Revisar conceitos de NAT IPv4 relevantes para o CCNA em uma topologia simples com dois roteadores Cisco, utilizando o PNETLab.

## Topologia e endereçamento

| Dispositivo/interface | Endereço IPv4 | Função |
|---|---|---|
| R0 Loopback0 | 10.0.0.1/8 | Endereço interno usado nos testes |
| R0 FastEthernet0/0 | 200.0.0.1/24 | Interface voltada para a rede externa |
| R1 FastEthernet0/0 | 200.0.0.10/24 | Destino de teste |
| Pool NAT dinâmico | 200.0.0.100–200.0.0.110/24 | Endereços usados na tradução dinâmica |

> Os endereços do pool devem estar reservados para NAT e não podem estar atribuídos a outros dispositivos. A topologia é didática; não representa uma faixa pública real destinada à Internet.

## Conceitos revisados

- **NAT estático:** mapeamento fixo de um endereço interno para um endereço global.
- **NAT dinâmico:** alocação de endereços globais a partir de um pool.
- **ACL padrão:** seleciona os endereços de origem que podem ser traduzidos; neste laboratório, ela é usada para seleção de NAT, não como controle de segurança.
- **Inside/Outside:** identifica os lados interno e externo da tradução.
- **Loopback como origem:** permite testar o tráfego com origem em um endereço diferente do IP da interface física.

## Configuração de referência — NAT estático

O mapeamento estático usado como exercício é:

```cisco
configure terminal
ip nat inside source static 10.0.0.1 200.0.0.2
interface loopback0
 ip nat inside
exit
interface fastethernet0/0
 ip nat outside
end
```

A tradução estática acima é um exemplo de configuração para o exercício. O endereço global `200.0.0.2` precisa estar reservado e ser alcançável na topologia. Para removê-la:

```cisco
configure terminal
no ip nat inside source static 10.0.0.1 200.0.0.2
end
```

## Configuração — NAT dinâmico

ACL para selecionar a rede interna:

```cisco
access-list 1 permit 10.0.0.0 0.255.255.255
```

Pool de endereços para tradução:

```cisco
ip nat pool PoolforNAT 200.0.0.100 200.0.0.110 netmask 255.255.255.0
```

Associação da ACL ao pool:

```cisco
ip nat inside source list 1 pool PoolforNAT
```

Definição dos lados NAT:

```cisco
interface loopback0
 ip nat inside
exit
interface fastethernet0/0
 ip nat outside
end
```

### O papel de cada comando

- `access-list 1 permit ...`: seleciona os IPs de origem que correspondem à regra de NAT.
- `ip nat pool ...`: define o intervalo de endereços que podem ser usados na tradução.
- `ip nat inside source list 1 pool PoolforNAT`: associa a seleção feita pela ACL ao pool.
- `ip nat inside` e `ip nat outside`: identificam as interfaces internas e externas.

## Testes e verificação

Gerar tráfego com a Loopback0 como origem:

```cisco
ping 200.0.0.10 source loopback0
```

Verificar traduções e estatísticas:

```cisco
show ip nat translations
show ip nat statistics
show access-lists 1
```

Durante o teste de NAT dinâmico, a tabela exibiu uma tradução ICMP de `10.0.0.1` (inside local) para `200.0.0.100` (inside global), com destino `200.0.0.10`. O ping teve sucesso de 80% (4 de 5 pacotes), portanto houve conectividade, mas também perda de um pacote durante esse teste.

## Observações importantes

- NAT não substitui roteamento: o destino precisa ser alcançável e o caminho de retorno deve funcionar.
- Para o NAT dinâmico funcionar, o pool precisa ter endereços livres.
- A ACL associada ao NAT seleciona tráfego para tradução; ela não bloqueia automaticamente o tráfego que não corresponde à regra.
- O primeiro ping pode perder um pacote durante a resolução ARP, mas isso não deve ser presumido sem novos testes.
- O laboratório foi usado para revisão de conceitos do CCNA. Não foi documentada validação de conectividade com a Internet.

## Comandos úteis para troubleshooting

```cisco
show ip interface brief
show running-config | include ip nat
show ip nat translations
show ip nat statistics
show access-lists 1
```
