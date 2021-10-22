# 🛠 Dev Firewall | Como configurar um firewall usando firewalld

Firewalld é um software de gerenciamento de firewall disponível para muitas distribuições Linux, que atua como um frontend para os sistemas de filtragem de pacotes nftables ou iptables do Linux. </br>

Neste guia, mostraremos como configurar um firewall firewalld para o seu servidor RHEL / CentOS 8 e abordaremos os fundamentos do gerenciamento do firewall com a firewall-cmdferramenta administrativa.

> Como configurar um firewall usando firewalld no  RHEL / CentOS 8

### Conceitos básicos em firewalld

Antes de começarmos a falar sobre como realmente usar o firewall-cmdutilitário para gerenciar a configuração do firewall, devemos nos familiarizar com alguns conceitos que a ferramenta apresenta.

### Zonas

O firewallddaemon gerencia grupos de regras usando entidades chamadas zonas . As zonas são conjuntos de regras que determinam qual tráfego deve ser permitido, dependendo do nível de confiança que você tem na rede. As interfaces de rede são atribuídas a uma zona para ditar o comportamento que o firewall deve permitir.</br>

Para computadores que podem se mover entre redes com frequência (como laptops), esse tipo de flexibilidade fornece um bom método de alterar suas regras, dependendo de seu ambiente. Você pode ter regras rígidas que proíbem a maior parte do tráfego ao operar em uma rede WiFi pública, enquanto permite restrições mais relaxadas quando conectado à sua rede doméstica. Para um servidor, essas zonas geralmente não são tão importantes porque o ambiente de rede raramente, ou nunca, muda.</br>

Independentemente de quão dinâmico seu ambiente de rede possa ser, ainda é útil estar familiarizado com a ideia geral por trás de cada uma das zonas predefinidas para firewalld. As zonas predefinidas firewalldsão, em ordem de menos confiável para mais confiável : </br>

* Block : O nível mais baixo de confiança. Todas as conexões de entrada são interrompidas sem resposta e apenas as conexões de saída são possíveis. </br>

* Block : Semelhante ao anterior, mas em vez de simplesmente eliminar as conexões, as solicitações de entrada são rejeitadas com uma mensagem icmp-host-prohibitedou icmp6-adm-prohibited.

* Public : representa redes públicas não confiáveis. Você não confia em outros computadores, mas pode permitir conexões de entrada selecionadas caso a caso.

* External : Redes externas caso você esteja usando o firewall como gateway. Ele é configurado para mascaramento de NAT para que sua rede interna permaneça privada, mas acessível.

* Internal : O outro lado da zona externa, usado para a parte interna de um gateway. Os computadores são bastante confiáveis ​​e alguns serviços adicionais estão disponíveis.

* Dmz : Usado para computadores localizados em uma DMZ (computadores isolados que não terão acesso ao resto da rede). Apenas certas conexões de entrada são permitidas.

* Work : Usado para máquinas de trabalho. Confie na maioria dos computadores da rede. Mais alguns serviços podem ser permitidos.

* Home : um ambiente doméstico. Geralmente implica que você confia na maioria dos outros computadores e que mais alguns serviços serão aceitos.

* Trusted : confia em todas as máquinas da rede. A mais aberta das opções disponíveis e deve ser usada com moderação.
