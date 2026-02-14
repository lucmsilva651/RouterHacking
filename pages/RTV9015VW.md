# Modem xDSL ASKEY RTV9015VW

Parte das informações de hardware e métodos de desbloqueio foram retirados do artigo do [O3 Labs](https://www.tripleoxygen.net/wiki/modem/rtv9015vw).

> [!WARNING]
> Só realize modificações neste dispositivo se ele for de sua propriedade. A maioria dos modems fornecidos pelos ISPs no Brasil é entregue em **comodato**, ou seja, não pertence ao usuário final. Alterações em equipamentos que não são seus podem gerar penalidades ou perda de suporte técnico.

---

## Tópicos

- [**Sobre o Modem**](#sobre-o-modem)
- [**Hardware**](#hardware)
  - [**Especificações de Hardware**](#especificações-do-hardware)
  - [**Foto do Hardware**](#foto-do-hardware)
- [**Software**](#software)
  - [**Desbloqueio das Configurações Avançadas**](#desbloqueio-das-configurações-avançadas)
  - [**Acesso SSH**](#acesso-ssh)

---

## Introdução

### Sobre o Modem

O **ASKEY RTV9015VW** é um modem xDSL (ADSL/VDSL) utilizado por diferentes provedores de internet no Brasil e também pela operadora Movistar em outros países. Possui duas variantes físicas: uma versão compacta e outra ligeiramente maior. Ambas compartilham a mesma arquitetura interna.

Na versão distribuída no Brasil, foi identificada uma vulnerabilidade no firmware inicial:

- O acesso ao **Wi-Fi** podia ser feito utilizando apenas o **endereço MAC da interface** como senha.
- Qualquer ferramenta de varredura Wi-Fi conseguia obter o MAC, facilitando o acesso não autorizado.

Essa falha foi corrigida em atualizações posteriores de firmware. Entretanto, a atualização removeu a página `saveconf.htm`, que permitia exportar e reimportar configurações avançadas.

---

## Hardware

### Especificações do Hardware

- **SoC**: Realtek RTL8685SF
- **Memória Flash**: MX25L6406E (8 MB)
- **Wireless 2.4 GHz**: RTL8192ER (802.11b/g/n)
- **DSL**: 1 porta (controlada pelo RTL8275)
- **FXS**: 1 porta (Microsemi Le9641PQC)
- **Serial Debug**: Presente
- **HPNA**: Não suportado

### Foto do Hardware

![Hardware RTV9015VW](https://www.tripleoxygen.net/wiki/_media/modem/img_20200722_211902.jpg)

- Créditos: [TripleOxygen](https://www.tripleoxygen.net)

---

## Software

### Flash layout

```
XDSL>show system flashlayout
Flash Layout:
Flash Start Address              : 0xb4000000   End Address      : 0xb4800000 Size   : 8MB
Flash Run Image Start Address    : 0xb4010000
Flash Backup Image Address       : 0xb4400000   Offset           : 4MB
Flash HW Config Address          : 0xb4005000   End Address      : 0xb4006000 Size   : 4KB
Flash Config Address             : 0xb47f0000   End Address      : 0xb4800000 Size   : 64KB
Flash Backup Config Address      : 0xb47e0000   End Address      : 0xb47f0000 Size   : 64KB
Flash file start Address         : 0xb43f8000   End Address      : 0xb4400000 Size   : 32KB
Flash unused start Address       : 0xb42e2000   End Address      : 0xb43f8000 Size   : 1112KB
Flash Possible Max Firmware      : 3997696[ 3904KB ]
Flash Defined  Max Firmware      : 4063232[ 3968KB ]
```

### Desbloqueio das Configurações Avançadas

> Estes passos permitem restaurar o acesso às páginas ocultas de configuração avançada do modem. Requer conhecimentos básicos em **edição de arquivos XML** e **operações de firmware**.

#### Passo 1 — Acesso inicial

- Abra no navegador: <http://192.168.15.1/padrao>
- Usuário: `support`
- Senha: senha padrão de login do modem (ex.: `simPutAq`)

#### Passo 2 — Exportar configurações

- Acesse: <http://192.168.15.1/saveconf.htm>
- Clique em **Save** e defina um nome para o arquivo exportado
- Faça uma **cópia de segurança** do arquivo antes de editar

#### Passo 3 — Editar o arquivo de configuração

- Abra o arquivo exportado em um editor de texto simples (ex.: **Bloco de Notas**)
- Localize a linha:

  ```xml
  <V N="PADRAO_WIFI_ONLY" V="0x1"/>
  ```

  Altere `0x1` para `0x0`.

- Localize a linha:

  ```xml
  <V N="ENABLE_ADVANCE_PAGE" V="0x0"/>
  ```

  Altere `0x0` para `0x1`.

- Salve o arquivo editado.

#### Passo 4 — Reimportar a configuração

- Volte à página: <http://192.168.15.1/saveconf.htm>
- Clique em **Escolher arquivo** e selecione o arquivo modificado
- Clique em **Upload**
- Aguarde o modem reiniciar automaticamente

#### Passo 5 — Verificação

- Após o reboot, acesse novamente: <http://192.168.15.1/padrao>
- Confirme se as páginas avançadas estão disponíveis

---

### Acesso SSH

> [!NOTE]
> A habilitação do SSH expõe funções administrativas internas do modem. Use com cautela. Não me responsabilizo por danos ou mau uso resultantes deste procedimento.

#### Passo 1 — Editar o arquivo de configuração

- Abra o mesmo arquivo que você usou para o desbloqueio no **Bloco de Notas**
- Localize:

  ```xml
  <V N="SSH_STATE" V="0x0"/>
  ```

  Altere `0x0` para `0x1`

- Localize:

  ```xml
  <chain N="SSH2AUTH_PWD">
  ```

  Anote as credenciais listadas (usuário/senha SSH) para uso futuro no ``sh``, se necessário

- Salve o arquivo

#### Passo 2 — Reimportar configuração

- Retorne a <http://192.168.15.1/saveconf.htm>
- Clique em **Escolher arquivo** e selecione o arquivo modificado
- Clique em **Upload**
- Aguarde o modem reiniciar

#### Passo 3 — Conectar via SSH

- Instale o **MobaXterm** (recomendado para compatibilidade)
- Execute o comando:

  ```bash
  ssh -legacy -c 3des-cbc support@192.168.15.1
  ```
  
- Senha: a mesma utilizada para o login no painel do modem
- Caso conectado com sucesso, você terá acesso ao shell do modem
