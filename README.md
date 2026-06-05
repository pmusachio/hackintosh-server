# 🖥️ Hackintosh Server (Dell Inspiron 5558)

Bem-vindo ao repositório do **Hackintosh Server**! Este projeto tem como objetivo transformar um notebook antigo Dell Inspiron 15 (5558) em um servidor macOS Monterey funcional e estável para uso em um Homelab. 

O repositório contém a pasta EFI perfeitamente lapidada (OpenCore) para garantir o boot nativo do macOS, além de (futuramente) conter todos os scripts e configurações de infraestrutura do servidor (Docker, automações, serviços, etc).

---

## 💻 Especificações do Hardware

- **Notebook:** Dell Inspiron 15 (Série 5000 / Modelo 5558)
- **Processador:** Intel Core i5-5200U (Broadwell)
- **Gráficos Integrados:** Intel HD Graphics 5500
- **Bootloader:** OpenCore 1.0.7 RELEASE
- **Sistema Operacional Alvo:** macOS Monterey (12.7.x)

---

## 🛠️ A Magia da EFI (Correções Específicas)

Se você tem esse mesmo notebook e está enfrentando Kernel Panics no instalador ou telas pretas ("borked"), saiba que o hardware da Dell e a arquitetura Broadwell exigem alguns "band-aids" cirúrgicos no arquivo `config.plist`.

Aqui estão os pilares de estabilidade desta EFI:

1. **A Maldição do RTC (OCSMC: SmcReadValue Panic):**
   Notebooks Dell mais antigos possuem incompatibilidades graves de leitura de memória RTC com o núcleo do macOS, causando travamentos instantâneos ao inicializar.
   - **Solução:** O quirk `DisableRtcChecksum = True` foi ativado no OpenCore para blindar essa leitura.

2. **Alocação de Memória Broadwell:**
   Processadores de 5ª Geração (Broadwell) tendem a falhar no handoff de inicialização.
   - **Solução:** O quirk `SetupVirtualMap = True` está ativado e corrige o mapa de memória UEFI antes do Kernel da Apple assumir o controle.

3. **Incompatibilidade de IRQ / ACPI:**
   O OpenCore 1.0.0+ é extremamente rigoroso com as tabelas ACPI. Mantivemos os SSDTs puros (`SSDT-PLUG-DRTNIA`, `SSDT-EC-LAPTOP` e `SSDT-PNLF`) sem emaranhados de patches Clover legados para garantir 100% de validação no `ocvalidate`.

---

## 🚀 Como Usar esta EFI

Se você quer reproduzir esse Hackintosh no seu Dell 5558:

1. **Prepare o Instalador Offline:** Crie o pendrive do macOS Monterey usando o comando nativo `createinstallmedia` em um Mac real. Evite instaladores "Recovery" de Windows.
2. **Copie a EFI:** Substitua a pasta `EFI` da partição oculta do seu pendrive por esta pasta `EFI` do repositório.
3. **Boot e Reset:** Ligue o notebook, entre no OpenCore pelo pendrive e faça um **Reset NVRAM**.
4. **BIOS Invisível?** Se após a instalação, a BIOS do Dell não reconhecer o disco UEFI interno:
   - Formate a partição EFI do SSD em FAT32 puro via terminal: `sudo newfs_msdos -F 32 -v EFI /dev/disk0s1`
   - Copie a pasta `EFI` novamente, entre na BIOS (F2) -> *Add Boot Option* -> Selecione `BOOTx64.efi`.

---

## 🏠 Roadmap do Servidor (Homelab)

*(Em construção)*

- [x] Boot estável do macOS Monterey.
- [ ] Configuração de Acesso Remoto (VNC / SSH).
- [ ] Otimização de energia para operação contínua com a tampa fechada (Sleep tweaks / Amphetamine).
- [ ] Instalação e configuração de infraestrutura Docker / Serviços do Homelab.
