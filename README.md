# Projeto de Laboratório: OPNsense Firewall/Router Virtualizado

Este repositório documenta a instalação e configuração de um firewall/router OPNsense em um ambiente de máquina virtual (VMware Workstation/Player), projetado para fins de estudo e experimentação de rede.

## 📝 Visão Geral do Projeto

Este projeto tem como objetivo principal:
* Instalar e configurar o OPNsense como um firewall/router em uma máquina virtual.
* Compreender a configuração de interfaces de rede (WAN e LAN) em um ambiente virtual.
* Explorar as funcionalidades básicas e avançadas do OPNsense.
* Manter um registro versionado da configuração do OPNsense no GitHub.

## 💻 Ambiente de Hardware e Software

* **Sistema Operacional Host:** [Windows 11 Pro]
* **Software de Virtualização:** VMware Workstation Player [ou Pro]
* **Máquina Virtual (VM):**
    * **Sistema Operacional:** OPNsense 25.1 (amd64)
    * **RAM Alocada:** [4GB]
    * **CPUs Alocadas:** [2 vCPUs]
    * **Disco Rígido:** [50GB]
* **Conectividade do Host:** Utiliza uma única interface de rede física (Wi-Fi ou Ethernet) no host para a conexão de internet da VM.
Este diagrama representa a estrutura de rede virtual configurada para o OPNsense com as interfaces e IPs atualizados:

---

**Detalhes das Interfaces e IPs:**

* **Seu Computador (Host):**
    * **Placa de Rede Física:** Obtém IP via DHCP da rede principal (internet).
    * **VMware Network Adapter VMnet1 (Host-only):**
        * IP: `192.168.123.254`
        * Gateway Padrão: `192.168.123.1`
        * Servidor DNS Preferencial: `192.168.123.1`

* **VM OPNsense:**
    * **em0 (WAN - VMware Bridged):**
        * IP: `192.168.202.1/24` (Estático)
        * *Nota: Para acesso à internet, esta rede `192.168.202.0/24` deve ser roteável a partir do seu roteador principal ou o `em0` configurado para DHCP.*
    * **em1 (LAN - VMware Host-only VMnet1):**
        * IP: `192.168.123.1/24`
        * DHCP Server: Faixa `192.168.123.100` a `192.168.123.200`
---

* **WAN (em0 - Bridged):** Conectada diretamente à rede física do host, configurada com IP estático (`192.168.202.1`). Para ter acesso à internet, esta rede `192.168.202.0/24` deve ser roteável a partir do seu roteador principal. **(Nota: Em ambientes domésticos, a WAN normalmente pega um IP via DHCP do roteador principal.)**
* **LAN (em1 - Host-only VMnet1):** Uma rede virtual isolada entre o OPNsense e o host (e outras VMs Host-only). O OPNsense atua como gateway (`192.168.123.1`) e servidor DHCP para essa rede.
* **Host-only Adapter (VMnet1):** Configurado no sistema operacional host para ter um IP na mesma sub-rede da LAN do OPNsense, permitindo o acesso à GUI e testes na rede interna.

## 🚀 Passos de Instalação e Configuração

Aqui estão os passos detalhados para replicar este ambiente, **ajustados para a sua configuração atual de interfaces e IPs**:

1.  **Preparação:**
    * Baixar e instalar o VMware Workstation Player.
    * Baixar a imagem ISO do OPNsense (versão `dvd` AMD64).
    * Verificar se a virtualização (VT-x/AMD-V) está habilitada na BIOS/UEFI do host.

2.  **Criação da VM no VMware:**
    * Criar uma nova VM (`Other -> FreeBSD 64-bit`).
    * Configurar RAM ([4GB]), CPUs ([2]).
    * Adicionar um disco virtual ([50GB]).
    * Configurar **duas interfaces de rede**:
        * Adaptador 1 (primeiro adicionado, será **em0**): `Bridged`.
        * Adaptador 2 (segundo adicionado, será **em1**): `Host-only` (selecionar `VMnet1`).
    * Anexar a ISO do OPNsense ao drive de CD/DVD da VM.

3.  **Instalação do OPNsense:**
    * Ligar a VM e bootar pela ISO.
    * Login no console (`installer` / `opnsense`).
    * Seguir o `Guided Installation`, selecionando o disco virtual.
    * **ATENÇÃO na atribuição das interfaces no console (Opção 1 no menu):**
        * `Enter the WAN interface name or 'a' for auto-detection:` Digite `em0`
        * `Enter the LAN interface name or 'a' for auto-detection:` Digite `em1`
    * Configurar o IP da **LAN (em1)** para `192.168.123.1/24` via console (opção `2`).
    * Configurar o IP da **WAN (em0)** para `192.168.202.1/24` via console (opção `2`). **(Nota: Se sua internet não funcionar, considere alterar a WAN para DHCP.)**
    * Habilitar DHCP para a LAN (faixa `192.168.123.100` a `192.168.123.150`).
    * Gerar novo certificado GUI (`y`) e restaurar defaults (`y`).
    * Remover a ISO da VM e reiniciar.

4.  **Configuração do Host-only Adapter (VMnet1) no Host:**
    * No sistema operacional host (Windows), acessar as "Conexões de Rede".
    * Localizar "VMware Network Adapter VMnet1".
    * Configurar IPv4 estático:
        * IP: `192.168.123.254`
        * Máscara: `255.255.255.0`
        * Gateway: `192.168.123.1`
        * DNS: `192.168.123.1`
    * Desabilitar e habilitar o adaptador VMnet1.

5.  **Acesso à Web GUI:**
    * Abrir o navegador e acessar `https://192.168.123.1`.
    * Aceitar o aviso de segurança do certificado.
    * Login inicial (`root` / `opnsense`).

6.  **Configurações Pós-Instalação na GUI:**
    * Completar o Wizard inicial (Hostname, DNS, Timezone).
    * **MUDAR A SENHA DO USUÁRIO `root` IMEDIATAMENTE.**
    * Instalar o plugin `os-vmtools` (System -> Firmware -> Plugins).

## ✅ Solução de Problemas Notáveis (e Aprendizados)

* **Conflito de IP na LAN (Inicial):** A interface LAN do OPNsense estava com `192.168.1.1`, causando conflito com o gateway do roteador principal. Solução: Alterar o IP da LAN para uma faixa não utilizada (ex: `192.168.123.1/24`) via console.
* **Interfaces Invertidas:** As interfaces `em0` e `em1` foram automaticamente atribuídas de forma invertida (WAN em `em0`, LAN em `em1`) em relação ao planejamento comum, mas a configuração foi ajustada para funcionar com essa atribuição.
* **"Conexão Recusada" ao Acessar a GUI (Histórico):** Mesmo após a configuração correta de IPs e adaptadores, o acesso à GUI era recusado, apesar do `ping` funcionar.
    * **Causa e Solução:** [**Aqui você VAI DESCER AS SUAS NOTAS SOBRE COMO VOCÊ REALMENTE RESOLVEU O PROBLEMA!** O que você fez que finalmente permitiu o acesso? Foi uma configuração no VMware, no OPNsense, um ajuste final? Seja o mais específico possível.]
    * **Aprendizado:** [Descreva o que você aprendeu com essa experiência de depuração. Ex: "A importância de verificar logs e o status dos serviços, e como regras de firewall podem impactar a conectividade da própria GUI, mesmo quando o ping funciona."]

## ⚙️ Configurações e Explorando Recursos

* **Configuração Inicial:** (Detalhes do que foi configurado no wizard ou após, ex: DNS, NTP, etc.)
* **Regras de Firewall:** (Descreva regras que você criou para testes, ex: "Bloqueio de sites via aliases", "Permissão para SSH externo", etc.)
* **Pacotes Instalados:** (Liste os plugins que você instalou e configurou)
    * `os-vmtools`: Para integração aprimorada com o VMware.
    * `os-adguardhome`: Para filtragem de DNS e bloqueio de anúncios.
    * `os-openvpn`: Configuração de servidor VPN para acesso remoto.
    * [Outros pacotes que você explorar]
* **Testes Realizados:** (criação de regras para comunicação com rede interna e privada de ip diferente, criação de regra para acesso apartir desse ip)

## 📸 Screenshots

![vm - opnsense](https://github.com/user-attachments/assets/d510409d-a0f9-4bfe-88b1-316b8684b5aa)
![interfaces](https://github.com/user-attachments/assets/2c768ca3-1cd9-4e88-9a7c-d72c00b1e8ba)
![interfaces - tabela arp](https://github.com/user-attachments/assets/103c32bf-2b44-44e1-a62d-af7976342a4a)

## 🤝 Contribuições

Sinta-se à vontade para sugerir melhorias ou compartilhar seus próprios aprendizados com o OPNsense.

## 📝 Licença

Este projeto é para fins educacionais e de estudo.
