# 🖥️ Hackintosh Server (Dell Inspiron 5558)

Bem-vindo ao repositório do **Hackintosh Server**! Este guia passo a passo ensina como transformar um notebook Dell Inspiron 15 (5558) em um servidor macOS Monterey funcional para uso em um Homelab.

Este repositório contém a pasta `EFI` otimizada com arquitetura preparada para o uso do OpenCore Legacy Patcher (OCLP), essencial para ativar placas de rede mais antigas no Monterey.

---

## 💻 Especificações do Hardware

- **Notebook:** Dell Inspiron 15 (Série 5000 / Modelo 5558)
- **Processador:** Intel Core i5-5200U (Broadwell)
- **Gráficos Integrados:** Intel HD Graphics 5500
- **Bootloader:** OpenCore 1.0.7 RELEASE
- **Sistema Operacional Alvo:** macOS Monterey (12.7.x)
- **Rede Cabeada:** Realtek PCIe FE Family Controller (Requer OCLP / RTL8100)
- **Rede Wi-Fi:** DELL Wireless 1707 / Atheros AR9565 (Requer OCLP / AirPortAtheros40)

---

## 🛠️ Guia Completo de Instalação (Via Terminal)

Para executar esses passos, você precisará ter acesso a um Mac (ou a outro Hackintosh) para preparar o pendrive de boot.

### Passo 1: Download do macOS Monterey
Abra o **Terminal** do Mac e digite o seguinte comando para baixar a imagem completa oficial da Apple do macOS Monterey (12.7.6):

```bash
softwareupdate --fetch-full-installer --full-installer-version 12.7.6
```
*O arquivo será baixado para a sua pasta de Aplicativos (`/Applications/Install macOS Monterey.app`).*

### Passo 2: Formatar o Pendrive
Conecte o seu pendrive (mínimo de 16GB). Descubra o número do disco dele rodando:

```bash
diskutil list
```
Localize o seu pendrive na lista (exemplo: `/dev/disk2` ou `/dev/disk3`). **Tenha absoluta certeza do número antes de prosseguir!**

Formate o pendrive para o formato exigido pelo instalador substituindo o `diskX` pelo número do seu pendrive:

```bash
diskutil eraseDisk JHFS+ "USB" /dev/diskX
```

### Passo 3: Criar o Instalador do macOS
Use a ferramenta oficial da Apple embutida no aplicativo que você baixou para injetar o macOS no pendrive:

```bash
sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/USB
```
*Digite a sua senha, pressione `Y` para confirmar e aguarde (pode demorar até 30 minutos).*

### Passo 4: Instalar o OpenCore (Pasta EFI)
O pendrive base do macOS não funciona sozinho em um PC. Precisamos injetar o "cérebro" do Hackintosh (nossa pasta `EFI`).

1. Baixe os arquivos deste repositório para o seu computador.
2. Monte a partição oculta do pendrive pelo Terminal:
   ```bash
   # Descubra o número da partição EFI do pendrive (ex: disk2s1)
   diskutil list
   sudo diskutil mount diskXs1
   ```
3. Um disco branco chamado `EFI` aparecerá na sua área de trabalho.
4. **Copie a pasta `EFI` deste repositório e cole dentro desse disco branco.**

### Passo 5: Configurar a BIOS do Dell 5558
Ligue o notebook pressionando **F2**:
- **SATA Operation:** AHCI
- **Secure Boot:** Disabled
- **Advanced Battery Charge:** Disabled
- **Fastboot:** Minimal

### Passo 6: Instalação no Dell
1. Dê boot pelo pendrive (Pressione **F12** e escolha a opção UEFI do USB).
2. No menu do OpenCore, escolha **Install macOS Monterey**.
3. Na tela principal, abra o **Utilitário de Disco (Disk Utility)**.
4. No topo esquerdo, clique em *Visualizar (View)* -> **Mostrar Todos os Dispositivos (Show All Devices)**.
5. Selecione a raiz do seu SSD interno (ex: APPLE SSD, KINGSTON, etc) e clique em **Apagar (Erase)**:
   - **Nome:** `Macintosh HD`
   - **Formato:** APFS
   - **Esquema:** Mapa de Partição GUID (GUID Partition Map)
6. Feche o Utilitário de Disco e prossiga com a instalação. O sistema irá reiniciar várias vezes. Escolha sempre a opção do SSD (`Macintosh HD`) no menu do OpenCore.

---

## 🚀 Pós-Instalação: Copiando a EFI e o Bug da BIOS

Após chegar na Área de Trabalho do macOS, você precisará copiar a EFI do pendrive para o SSD, para o notebook conseguir ligar sozinho.

1. Monte a EFI do Pendrive e a EFI do SSD interno no Terminal.
2. Copie a pasta `EFI` do pendrive e cole na partição do SSD interno.

**⚠️ Solução para o Bug crônico da Placa Dell ("No Bootable Devices Found")**
A BIOS do Dell frequentemente ignora a pasta EFI copiada via macOS. Para forçar a leitura, abra o Terminal e formate a partição EFI do SSD em FAT32 puro:

```bash
# CUIDADO: disk0s1 costuma ser a EFI do SSD. Verifique antes com diskutil list!
sudo newfs_msdos -F 32 -v EFI /dev/disk0s1
```
Após o comando, copie a pasta `EFI` do pendrive novamente para o SSD vazio. Reinicie, entre na BIOS (F2) -> *Add Boot Option* -> Selecione o arquivo `BOOTx64.efi` e coloque-o no topo da lista.

---

## 🌐 Ativando as Placas de Rede Legadas (OCLP)

Como este projeto utiliza placas de rede jurássicas incompatíveis com o Monterey (Realtek RTL810x e Wi-Fi Atheros AR9565), **esta EFI foi construída com o SIP (System Integrity Protection) e o Secure Boot desligados.**

Para trazer essas placas de volta à vida:
1. Baixe o aplicativo **OpenCore Legacy Patcher (OCLP)**.
2. Abra o aplicativo.
3. Clique em **Post-Install Root Patch**.
4. O programa vai detectar os frameworks antigos necessários. Inicie o "Start Root Patching", aguarde a barra de progresso e reinicie o computador.

> [!WARNING]
> Ao usar o OCLP em um Servidor, evite realizar grandes atualizações automáticas do macOS via Preferências do Sistema, pois as atualizações apagarão o Root Patch e o servidor perderá a conexão de rede. Atualize sempre manualmente reaplicando o OCLP.

---

## ⌨️ Configuração ABNT2 (Trackpad e Teclado)
- **Trackpad (Sem Clique?):** Esta EFI desativa o driver genérico de mouse (`VoodooPS2Mouse.kext`) para dar espaço ao driver nativo de Trackpad. Vá em Ajustes do Sistema -> Trackpad e ative "Clicar com um toque".
- **Teclado:** Em Ajustes -> Teclado -> Fontes de Entrada, adicione `Brasileiro - ABNT2` e remova o `EUA` padrão.
