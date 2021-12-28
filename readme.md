# 🛠 Dev Firewall | Configurar um firewall usando firewalld

Firewalld é um software de gerenciamento de firewall disponível para muitas distribuições Linux, que atua como um frontend para os sistemas de filtragem de pacotes nftables ou iptables do Linux. </br>

Neste guia, mostraremos como configurar um firewall firewalld para o seu servidor RHEL / CentOS 8 e abordaremos os fundamentos do gerenciamento do firewall com a firewall-cmdferramenta administrativa.

> Como configurar um firewall usando firewalld no  RHEL / CentOS 8

### Conceitos básicos em firewalld

Antes de começarmos a falar sobre como realmente usar o firewall-cmdutilitário para gerenciar a configuração do firewall, devemos nos familiarizar com alguns conceitos que a ferramenta apresenta.

### Zonas

O firewalld daemon gerencia grupos de regras usando entidades chamadas zonas.</br></br>
As zonas são conjuntos de regras que determinam qual tráfego deve ser permitido, dependendo do nível de confiança que você tem na rede.</br></br>
As interfaces de rede são atribuídas a uma zona para ditar o comportamento que o firewall deve permitir.</br>

Para computadores que podem se mover entre redes com frequência (como laptops), esse tipo de flexibilidade fornece um bom método de alterar suas regras, dependendo de seu ambiente.</br></br>
Você pode ter regras rígidas que proíbem a maior parte do tráfego ao operar em uma rede WiFi pública, enquanto permite restrições mais relaxadas quando conectado à sua rede doméstica.</br></br>
Para um servidor, essas zonas geralmente não são tão importantes porque o ambiente de rede raramente, ou nunca, muda.</br>

Independentemente de quão dinâmico seu ambiente de rede possa ser, ainda é útil estar familiarizado com a ideia geral por trás de cada uma das zonas predefinidas para firewalld.</br></br>
As zonas predefinidas firewalldsão, em ordem de menos confiável para mais confiável :</br>

- [x] Drop : O nível mais baixo de confiança.</br>
Todas as conexões de entrada são interrompidas sem resposta e apenas as conexões de saída são possíveis. </br>

```Drop
<?xml version="1.0" encoding="utf-8"?>
<zone target="DROP">
  <short>Drop</short>
  <description>Unsolicited incoming network packets are dropped. 
  Incoming packets that are related to outgoing network connections are accepted. 
  Outgoing network connections are allowed.</description>
</zone>
```

- [x] Block : Semelhante ao anterior, mas em vez de simplesmente eliminar as conexões, as solicitações de entrada são rejeitadas com uma mensagem icmp-host-prohibitedou icmp6-adm-prohibited.

```Block
<?xml version="1.0" encoding="utf-8"?>
<zone target="%%REJECT%%">
  <short>Block</short>
  <description>Unsolicited incoming network packets are rejected. 
  Incoming packets that are related to outgoing network connections are accepted. 
  Outgoing network connections are allowed.</description>
</zone>
```

- [x] Public : Representa redes públicas não confiáveis.</br>
Você não confia em outros computadores, mas pode permitir conexões de entrada selecionadas caso a caso.

```Public
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. 
  Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
</zone>
```

- [x] External : Redes externas caso você esteja usando o firewall como gateway.</br>
Ele é configurado para mascaramento de NAT para que sua rede interna permaneça privada, mas acessível.

```External
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>External</short>
  <description>For use on external networks. You do not trust the other computers on networks to not harm your computer. 
  Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <masquerade/>
</zone>
```

- [x] Internal : O outro lado da zona externa, usado para a parte interna de um gateway.</br>
Os computadores são bastante confiáveis e alguns serviços adicionais estão disponíveis.


- [x] Dmz : Usado para computadores localizados em uma DMZ (computadores isolados que não terão acesso ao resto da rede).</br>
Apenas certas conexões de entrada são permitidas.

- [x] Work : Usado para máquinas de trabalho. Confie na maioria dos computadores da rede.</br>
Mais alguns serviços podem ser permitidos.

- [x] Home : Um ambiente doméstico. Geralmente implica que você confia na maioria dos outros computadores e que mais alguns serviços serão aceitos.

```Home
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Home</short>
  <description>For use in home areas. You mostly trust the other computers on networks to not harm your computer. 
  Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="mdns"/>
  <service name="samba-client"/>
  <service name="dhcpv6-client"/>
</zone>
```

- [x] Trusted : Confia em todas as máquinas da rede. A mais aberta das opções disponíveis e deve ser usada com moderação.

```Trusted
<?xml version="1.0" encoding="utf-8"?>
<zone target="ACCEPT">
  <short>Trusted</short>
  <description>All network connections are accepted.</description>
</zone>
```

> Para usar o firewall, podemos criar regras e alterar as propriedades de nossas zonas e, em seguida, atribuir nossas interfaces de rede às zonas mais apropriadas.

### Permanência de regra

No firewalld, as regras podem ser aplicadas ao conjunto de regras de tempo de execução atual ou tornar-se permanentes.</br>
Quando uma regra é adicionada ou modificada, por padrão, apenas o firewall em execução no momento é modificado.</br>
Após a próxima reinicialização - ou recarga do firewalldserviço - apenas as regras permanentes permanecerão.

A maioria das firewall-cmdoperações pode receber um --permanentsinalizador para indicar que as alterações devem ser aplicadas à configuração permanente.</br>

Além disso, o firewall em execução no momento pode ser salvo na configuração permanente com o firewall-cmd --runtime-to-permanentcomando.

Essa separação entre o tempo de execução e a configuração permanente significa que você pode testar as regras com segurança em seu firewall ativo e, em seguida, recarregar para reiniciar se houver problemas.

### Instalando e habilitando firewalld
