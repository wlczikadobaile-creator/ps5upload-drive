# PS5Upload Drive: backup automático de saves no Google Drive

Versão modificada do **[PS5Upload](https://github.com/phantomptr/ps5upload)**, criado por
**[phantomptr](https://github.com/phantomptr)**. Todo o crédito do PS5Upload é dele. Esta
versão só acrescenta o backup dos saves no Google Drive, feito direto pelo PS5, sem PC.

É baseada na versão oficial **v11.0.1** e segue a mesma licença, a **GPL-3.0**. O código das
mudanças está em [`ps5upload-gdrive-v11.0.1.patch`](ps5upload-gdrive-v11.0.1.patch).

> Não é uma versão oficial. Se der problema, abra uma issue **aqui**, não no repositório original.

**Por que não está no PS5Upload oficial?** O recurso foi oferecido ao autor
([issue #674](https://github.com/phantomptr/ps5upload/issues/674)). Ele preferiu não incluir
porque o PS5Upload não guarda credenciais de ninguém, e aqui o acesso ao Google Drive (um token)
fica salvo no PS5. Ele incentivou manter esta versão em paralelo. Se você prefere não deixar
nenhum login no console, use a alternativa que ele sugeriu: no app do PC, **Save Data ›
Automatic backups** numa pasta que o Google Drive sincroniza. Essa opção precisa do PC ligado com
o app aberto.

## O que ela faz

- **Backup automático** a cada 15 min, 30 min, 1 h, 2 h ou 6 h, mesmo com o jogo aberto. O save
  é copiado primeiro no próprio PS5 (rápido) e depois enviado ao Drive. Só vai o que mudou.
- No Drive fica `PS5 Saves/<perfil>/<jogo>/<jogo>_<data>.tar`, com os 10 mais recentes de cada jogo.
- **Restaurar** pelo Minion: Saves › o jogo › "Ver os backups no Google Drive" › Restaurar.
  O save atual é copiado antes.
- **Perfis**: junto com os saves vai `PS5 Saves/perfis_<data>.txt`, com o **ID da conta** de cada
  perfil. Ele é enviado se ainda não existir no Drive ou se algo mudou.
- **Restaurar perfil**: se você perder um perfil, crie-o de novo. No cartão do Google Drive,
  "Restaurar perfil" devolve a ele o ID da conta antiga, e os saves voltam por "De outros perfis".
- O ícone na tela inicial vira **PS5Upload Drive** (azul, com nuvem), para não confundir com o
  original.
- Acesso só aos arquivos que o próprio app cria no seu Drive (permissão `drive.file`). HTTPS
  com os certificados do Google conferidos.

## Como usar

### 1. Carregar o payload

Envie o **`ps5upload-gdrive.elf`** (está em [Releases](../../releases)) como você já envia
payloads: Payload Manager, porta 9021 etc. Use **no lugar** do ps5upload original, nunca os dois
juntos, porque eles usam as mesmas portas.

Se o seu loader tem carregamento automático (PLDMGR, etaHEN), coloque o `ps5upload-gdrive.elf` na
sequência no lugar do original.

### 2. Criar o acesso ao Google (uma vez só, de graça)

O app usa um cliente OAuth **seu**: nenhum dado passa por servidor de terceiros.

1. Entre em <https://console.cloud.google.com/> e crie um projeto.
2. **APIs e serviços › Biblioteca**: ative a **Google Drive API**.
3. **Google Auth Platform › Branding**: preencha o nome do app e o seu e-mail. **Não** envie
   logotipo.
4. **Público-alvo**: tipo **Externo**. Em **Usuários de teste**, adicione o seu Gmail.
   - Opcional: para o login não expirar a cada 7 dias, clique em **Publicar app**. O Google pede
     links de página inicial e de Política de Privacidade: dá para hospedar os dois de graça no
     GitHub Pages.
5. **Clientes › Criar cliente**: tipo **"TVs e dispositivos de entrada limitados"**. Baixe o
   **JSON** do cliente.

### 3. Conectar no PS5

1. Abra o **PS5Upload Drive** na tela inicial, ou, no PC ou celular, `http://<ip-do-ps5>:9180`
   (ative o acesso pela rede nas configurações do Minion).
2. Vá em **Saves**. No cartão **Backup no Google Drive**:
   - **Carregar o arquivo JSON**: escolha o arquivo baixado no passo 2.5 (no PC ou celular), ou
   - **Ler client_secret.json do USB**: com o `client_secret_….json` na raiz de um pendrive, ou
   - digite o ID do cliente e depois a chave secreta (começa com `GOCSPX-`).
3. Clique em **Conectar o Google Drive** e escaneie o **QR code** com o celular. Entre na sua
   conta e permita o acesso.
4. Pronto: o backup automático liga sozinho. Para fazer um na hora, use **Fazer backup agora**.

### Limitações

- O backup só roda **enquanto o payload estiver carregado**. Depois de reiniciar o PS5, faça o
  jailbreak e carregue o ELF de novo (ou use o carregamento automático).
- Os saves do PS5 são **criptografados pelo próprio console**. Os backups voltam no **mesmo PS5**,
  no mesmo perfil ou num perfil refeito com o mesmo ID de conta. Para outro PS5, use "Save as a
  file for a friend or another PS5" do app do PC.
- O relógio do PS5 precisa estar certo, porque os certificados HTTPS dependem dele.

## Compilar

1. Baixe o código oficial da **v11.0.1**:
   <https://github.com/phantomptr/ps5upload/archive/refs/tags/v11.0.1.tar.gz>
2. Aplique o patch: `patch -p1 < ps5upload-gdrive-v11.0.1.patch`
3. Extraia [`mbedtls-3.6.2-subset.tar.gz`](mbedtls-3.6.2-subset.tar.gz) em `payload/third_party/`.
   É o mbed TLS 3.6.2, sem mudanças, Apache-2.0.
4. Copie [`webui_tile_icon0.png`](webui_tile_icon0.png) por cima de `payload/assets/webui_tile_icon0.png`.
5. `make payload`, com o PS5 Payload SDK, como no projeto original.

---

## English

A modified build of **[PS5Upload](https://github.com/phantomptr/ps5upload)** by
**[phantomptr](https://github.com/phantomptr)**, based on upstream **v11.0.1** (GPL-3.0). It adds
automatic save backups to the user's own Google Drive, run by the payload itself: no PC is needed,
and it works while a game is running.

What it adds:
- Local snapshot first, then upload of changed saves.
- Restore from the Minion page.
- A profiles file with each profile's account ID.
- "Restore profile", which gives a re-created profile its old account ID.
- A home-screen tile renamed "PS5Upload Drive".

Sign-in is Google's OAuth device flow with a QR code, using the user's own "TVs and Limited Input"
client and the `drive.file` scope. TLS is mbed TLS 3.6.2, pinned to Google's root CAs.

Not an official release: please report problems here, not upstream. The feature was offered
upstream ([#674](https://github.com/phantomptr/ps5upload/issues/674)) and declined, because
PS5Upload doesn't store anyone's credentials and this build keeps a Google token on the console.
The author encouraged keeping it going as a separate build. If you'd rather keep no sign-in on
the console, use the alternative the author suggested: in the PC app, point Save Data › Automatic
backups at a folder that Google Drive syncs (this needs the PC app running). The full change is
[`ps5upload-gdrive-v11.0.1.patch`](ps5upload-gdrive-v11.0.1.patch) against the v11.0.1 tag.
