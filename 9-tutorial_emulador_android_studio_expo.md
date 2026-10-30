# Tutorial 9 — Configurar o emulador do Android Studio e usar no projeto Expo
### ARA0089 — Programação para Dispositivos Móveis em Android

Vamos criar um celular Android virtual e executar nele o projeto **React Native com Expo** desenvolvido nas aulas. O exemplo usa o **MeuAppFirebase**, mas você pode substituir pelo nome do seu aplicativo.

O roteiro principal usa **Windows 11**. Os comandos específicos de **macOS** e **Linux** aparecem em blocos separados e identificados. Execute somente o bloco do seu sistema.

**Ao final, você conseguirá:**

- criar e iniciar um dispositivo virtual Android;
- abrir seu projeto no emulador com Expo Go;
- acompanhar alterações usando Fast Refresh;
- executar um build de desenvolvimento quando o projeto exigir código nativo;
- instalar um APK no emulador e diagnosticar falhas de conexão.

> Execute os comandos na pasta do **aplicativo**, onde está o `package.json`. Este repositório contém os roteiros das aulas, não o código do app.

## 1. Pré-requisitos e escolha do fluxo

Tenha Node.js compatível com seu Expo SDK, dependências do projeto e Android Studio com SDK configurados. Para instalar essas ferramentas, siga o roteiro do seu sistema:

- [Tutorial 8 — Windows 11](8-tutorial_build_apk_android_studio.md).
- [Tutorial 8.1 — macOS](8.1-tutorial_build_apk_android_studio_macos.md).
- [Tutorial 8.2 — Linux](8.2-tutorial_build_apk_android_studio_linux.md).

**Não é necessário gerar um APK de release antes desta aula.** Para usar Expo Go no emulador, também não é necessário gerar a pasta `android/` nem abrir um projeto nativo na IDE.

| Fluxo | Quando usar | Precisa compilar o app localmente? |
|---|---|---|
| Expo Go | Projeto com bibliotecas disponíveis no Expo Go e SDK compatível | Não |
| Build de desenvolvimento | Projeto com dependências ou configurações nativas próprias | Sim, ao criar ou alterar a parte nativa |
| APK de release | Conferir a versão de entrega sem Metro | O APK precisa ter sido gerado anteriormente |

Comece pelo **Expo Go**, caso seu projeto seja compatível. A CLI ajuda a abrir o projeto no dispositivo virtual, conforme o [guia de emulador do Expo](https://docs.expo.dev/workflow/android-studio-emulator/).

## 2. Conferir o SDK e a aceleração

No Android Studio, abra **More Actions > SDK Manager** na tela inicial ou **Tools > SDK Manager** com um projeto aberto. Na aba **SDK Tools**, confira:

- **Android Emulator**;
- **Android SDK Platform-Tools**, que fornece o `adb`;
- **Android SDK Command-line Tools (latest)**, se utilizar as ferramentas de terminal.

Instale os componentes faltantes e anote **Android SDK Location**. Os tutoriais 8, 8.1 e 8.2 explicam como configurar `ANDROID_HOME` e adicionar `platform-tools` e `emulator` ao caminho do terminal.

### Windows 11

Confira no **Gerenciador de Tarefas > Desempenho > CPU** se a virtualização está habilitada. Se necessário, habilite Intel VT-x/AMD-V no BIOS/UEFI. Em **Ativar ou desativar recursos do Windows**, habilite **Windows Hypervisor Platform** e reinicie quando solicitado.

### macOS

Use uma imagem **ARM64** em Apple Silicon ou **x86_64** em Mac Intel. O emulador utiliza o mecanismo de virtualização disponível no macOS.

### Linux

Configure KVM e permissões do usuário conforme o tutorial 8.2. Verifique se `/dev/kvm` existe e está acessível ao seu usuário.

Para diagnosticar a aceleração, depois de configurar o caminho das ferramentas:

**Windows — PowerShell:**

```powershell
emulator -accel-check
```

**macOS — Terminal:**

```bash
emulator -accel-check
```

**Linux — Terminal:**

```bash
emulator -accel-check
```

Confira detalhes e requisitos no [guia oficial de aceleração do emulador](https://developer.android.com/studio/run/emulator-acceleration). Em computadores sem recursos suficientes, use um celular físico para acompanhar as aulas.

## 3. Criar o celular virtual no Device Manager

Um **AVD (Android Virtual Device)** reúne o perfil do aparelho, a imagem Android e suas configurações.

1. Na tela inicial da IDE, abra **More Actions > Virtual Device Manager**. Com um projeto aberto, use **Tools > Device Manager**.
2. Clique em **Create Virtual Device** ou no botão **+**.
3. Escolha um perfil de telefone, por exemplo um Pixel disponível.
4. Selecione uma imagem Android estável que atenda ao Android mínimo do aplicativo.
5. Use **x86_64** em Windows/Linux com processador Intel/AMD e em Mac Intel; use **arm64-v8a** em Mac Apple Silicon.
6. Baixe a imagem se necessário. Para serviços Google, escolha **Google APIs**; para ter Play Store, escolha uma imagem **Google Play** e um perfil compatível.
7. Nomeie o dispositivo como **AulaAndroid** e conclua em **Finish**.

Não altere o `compileSdk` do projeto para igualá-lo à imagem: a API do emulador representa o Android em execução, enquanto `compileSdk` é uma configuração de compilação.

Referência: [criação e gerenciamento de dispositivos virtuais](https://developer.android.com/studio/run/managing-avds).

## 4. Iniciar e conferir o emulador

No Device Manager, clique no botão de iniciar ao lado de **AulaAndroid**. Aguarde aparecer a tela inicial do Android e desbloqueie a tela.

Abra um terminal novo e confira:

**Windows — PowerShell:**

```powershell
adb devices
```

**macOS — Terminal:**

```bash
adb devices
```

**Linux — Terminal:**

```bash
adb devices
```

Exemplo de resultado:

```text
List of devices attached
emulator-5554    device
```

`emulator-5554` é um exemplo; use o identificador mostrado no seu computador. Aguarde se o dispositivo estiver `offline` durante a inicialização.

Você pode deixar o emulador aberto enquanto edita o JavaScript no VS Code. No fluxo Expo Go, o Android Studio serve para gerenciar o aparelho virtual.

## 5. Preparar o seu projeto

Entre na pasta do aplicativo e confira as dependências.

**Windows — PowerShell:**

```powershell
cd C:\Projetos\MeuAppFirebase
node --version
npm --version
npx expo-doctor
npx expo install --check
```

**macOS — Terminal:**

```bash
cd "$HOME/Projetos/MeuAppFirebase"
node --version
npm --version
npx expo-doctor
npx expo install --check
```

**Linux — Terminal:**

```bash
cd "$HOME/Projetos/MeuAppFirebase"
node --version
npm --version
npx expo-doctor
npx expo install --check
```

Substitua os caminhos pelos do seu computador. Se acabou de baixar o projeto, instale antes as dependências: use `npm ci` com um `package-lock.json` consistente ou `npm install` se não houver lockfile.

Se o PowerShell bloquear os scripts `npm.ps1` ou `npx.ps1`, execute `npm.cmd` e `npx.cmd` nos comandos correspondentes.

Para o **MeuAppFirebase**, confira o `.env.local` na raiz, conforme o [tutorial 6](6-tutorial_autenticacao_firebase_email_senha.md). O emulador usa a configuração do mesmo projeto; não precisa de outro projeto Firebase apenas por ser um dispositivo virtual.

## 6. Abrir o projeto no emulador com Expo Go

Com o emulador iniciado, execute na raiz do aplicativo:

**Windows — PowerShell:**

```powershell
npx expo start --go
```

**macOS — Terminal:**

```bash
npx expo start --go
```

**Linux — Terminal:**

```bash
npx expo start --go
```

Quando o Metro estiver pronto, pressione **`a` no terminal** para abrir no Android. Se a CLI solicitar instalar o Expo Go, confirme e aguarde. Não é necessário escanear QR Code no emulador.

Se houver vários dispositivos, pressione **`Shift+A`** para selecionar o desejado. A opção `--go` escolhe explicitamente Expo Go, mesmo que o projeto tenha `expo-dev-client` instalado.

Se surgir incompatibilidade de SDK, use uma versão compatível de Expo Go para Android, disponível em [expo.dev/go](https://expo.dev/go), ou siga o fluxo de development build da seção 9. Confira antes as versões do projeto; atualizar o SDK exige revisar dependências.

Os atalhos e opções estão documentados na [referência da Expo CLI](https://docs.expo.dev/more/expo-cli/).

## 7. Trabalhar no projeto durante a aula

1. Deixe o emulador e o terminal do Metro abertos.
2. Abra o projeto no VS Code.
3. Altere um texto visível em `App.js` ou na tela inicial que seu app utiliza.
4. Salve e observe a atualização no emulador.
5. No terminal do Expo, use **`r`** para recarregar ou **`m`** para abrir o menu de desenvolvimento.

O Fast Refresh costuma aplicar mudanças sem reinstalar o app. Algumas alterações exigem recarregar a tela; mudanças em dependências/configurações nativas exigem um novo build de desenvolvimento.

Se alterar variáveis do Firebase, pare o Metro com `Ctrl+C` e reinicie usando o comando do seu sistema:

**Windows — PowerShell:**

```powershell
npx expo start --go --clear
```

**macOS — Terminal:**

```bash
npx expo start --go --clear
```

**Linux — Terminal:**

```bash
npx expo start --go --clear
```

Pressione `a` novamente. Não é necessário apagar o emulador para limpar o cache do Metro.

### O que testar no MeuAppFirebase

- Login com uma conta de teste e mensagem ao informar dados inválidos.
- Cadastro de usuário, navegação e saída da conta.
- Cadastro, consulta, alteração e exclusão de um produto de teste.
- Campos com teclado aberto, rolagem e botão Voltar do Android.
- Mensagens quando o acesso à internet estiver indisponível.

Use dados de teste no Firebase. Se o app usa SQLite, confira também se os registros permanecem após fechar e abrir o aplicativo. O armazenamento do Expo Go e o do seu APK próprio são separados; não espere encontrar automaticamente os mesmos dados locais nos dois.

## 8. Conexão com Metro e serviços locais

### 8.1. Metro não conecta

Confirme que o emulador aparece em `adb devices` e que o Metro continua em execução. Se necessário, em um **segundo terminal**, crie o redirecionamento para o dispositivo correto:

**Windows — PowerShell:**

```powershell
adb -s emulator-5554 reverse tcp:8081 tcp:8081
```

**macOS — Terminal:**

```bash
adb -s emulator-5554 reverse tcp:8081 tcp:8081
```

**Linux — Terminal:**

```bash
adb -s emulator-5554 reverse tcp:8081 tcp:8081
```

Substitua o serial pelo resultado de `adb devices`. O exemplo considera Metro na porta 8081. Se ele usar outra porta, ajuste os dois números. Volte ao terminal do Expo e abra o projeto novamente.

Confira também firewall, VPN e proxy, permitindo a conexão necessária na sua rede de desenvolvimento. Não desative o firewall inteiro para testar o app.

### 8.2. API que roda no computador

Dentro do emulador, `localhost` aponta para o **próprio Android virtual**. Para alcançar um serviço do computador, o Android Emulator oferece o endereço especial **`10.0.2.2`**.

Exemplo: uma API acessível no computador por `http://localhost:3000` pode ser acessada pelo app em `http://10.0.2.2:3000`, desde que o serviço e o firewall permitam. Requisições HTTP sem TLS também podem depender da política de segurança do aplicativo.

Esse endereço é para o emulador Android; não deve substituir as URLs HTTPS do Firebase. Veja a [documentação de rede do Android Emulator](https://developer.android.com/studio/run/emulator-networking).

## 9. Alternativa: build de desenvolvimento no emulador

Use este fluxo se o aplicativo exige bibliotecas nativas que o Expo Go não contém. Ele modifica o projeto: salve seu trabalho e faça commit ou backup antes.

Para compilar e fazer deploy local, use **JDK 17**, conforme a configuração do tutorial 8 do seu sistema. Antes dos comandos abaixo, confira `java -version` e `javac -version`; ambos devem mostrar **17.x**. Depois de gerar `android/`, confira também `gradlew --version`, como explicado na seção 11 dos tutoriais de build. No Android Studio, selecione o mesmo JDK 17 em **Gradle JDK**.

O Expo Go da seção 6 não exige compilar seu app com Java. Esta orientação aplica-se ao build nativo de desenvolvimento e ao release.

Execute na raiz do app:

**Windows — PowerShell:**

```powershell
npx expo install expo-dev-client
npx expo run:android --device
```

**macOS — Terminal:**

```bash
npx expo install expo-dev-client
npx expo run:android --device
```

**Linux — Terminal:**

```bash
npx expo install expo-dev-client
npx expo run:android --device
```

Selecione **AulaAndroid** quando solicitado. A CLI gera `android/` se necessário, compila, instala e inicia o fluxo de desenvolvimento. A primeira compilação pode baixar ferramentas adicionais. Não use `prebuild --clean` automaticamente para corrigir erros: ele pode apagar alterações nativas.

Nos próximos dias, com o build já instalado, inicie somente o Metro:

**Windows — PowerShell:**

```powershell
npx expo start --dev-client
```

**macOS — Terminal:**

```bash
npx expo start --dev-client
```

**Linux — Terminal:**

```bash
npx expo start --dev-client
```

Pressione `a`. Recompile com `npx expo run:android --device` quando alterar bibliotecas ou configurações nativas. Esse build serve ao desenvolvimento e normalmente depende do Metro; para o APK de entrega, siga o tutorial 8 correspondente.

Referência: [build de desenvolvimento local com Expo](https://docs.expo.dev/guides/local-app-development/).

## 10. Testar um APK pronto no emulador

Para conferir o APK de release gerado nos tutoriais anteriores, instale pelo caminho real do arquivo:

**Windows — PowerShell:**

```powershell
adb -s emulator-5554 install -r "C:\Projetos\MeuAppFirebase\android\app\release\app-release.apk"
```

**macOS — Terminal:**

```bash
adb -s emulator-5554 install -r "$HOME/Projetos/MeuAppFirebase/android/app/release/app-release.apk"
```

**Linux — Terminal:**

```bash
adb -s emulator-5554 install -r "$HOME/Projetos/MeuAppFirebase/android/app/release/app-release.apk"
```

Substitua o serial e o caminho conforme seu ambiente. O `-r` reinstala preservando dados quando pacote e assinatura são compatíveis. O APK também precisa conter código nativo compatível com a arquitetura do emulador.

Abra pelo ícone do próprio aplicativo. Para testar o release, pare o Metro e confira que o app continua abrindo. Aplicativos dependentes de Firebase ainda precisam de internet.

## 11. Problemas comuns

| Problema | O que verificar |
|---|---|
| `adb` ou `emulator` não encontrado | Confira SDK, `ANDROID_HOME`, caminhos e reabra o terminal |
| Nenhum dispositivo em `adb devices` | Inicie o AVD e aguarde o Android terminar de carregar |
| Mais de um dispositivo conectado | Use `Shift+A` no Expo ou `adb -s SERIAL` |
| Emulador lento ou não inicia | Confira virtualização, arquitetura da imagem, RAM e espaço livre |
| AVD trava ao restaurar | Pare o emulador e tente **Cold Boot Now**, no menu do dispositivo |
| Estado do AVD continua corrompido | **Wipe Data** restaura o aparelho virtual e apaga seus apps e dados; use somente se puder perdê-los |
| Expo Go incompatível | Confira o SDK e a versão do Expo Go ou use development build |
| Projeto abre no cliente errado | Use `--go` para Expo Go ou `--dev-client` para development build |
| Não conecta ao Metro | Confira servidor, serial, porta, redirecionamento, VPN e firewall |
| API local não responde | Use `10.0.2.2` para alcançar o computador e confira porta/política HTTP |
| Firebase retorna erro | Confira `.env.local`, internet, autenticação e regras de segurança |
| `INSTALL_FAILED_NO_MATCHING_ABIS` | A arquitetura do APK não corresponde à imagem do emulador |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | Já existe app com mesmo pacote e outra assinatura; use a chave original |
| App instalado fecha ao abrir | Abra **Logcat** na IDE, selecione o processo e procure a exceção |

O emulador ajuda no desenvolvimento, mas câmera, desempenho e outros recursos também devem ser conferidos em um aparelho físico antes da entrega.

## 12. Rotina e atividade de verificação

Para as próximas aulas:

1. Inicie **AulaAndroid** pelo Device Manager.
2. Entre na pasta do aplicativo.
3. Inicie o Expo no fluxo escolhido: `--go` ou `--dev-client`.
4. Pressione `a`, altere o código e teste.
5. Ao encerrar, pare o Metro com `Ctrl+C` e feche o emulador.

**Atividade:** execute o projeto no emulador, altere um texto visível e registre uma captura da tela. No MeuAppFirebase, crie um produto de teste, altere-o e exclua-o. Em projetos SQLite, confirme a persistência após reabrir. Anote qual fluxo utilizou: Expo Go, development build ou APK de release.

- [ ] AVD criado com arquitetura compatível.
- [ ] Android inicializa e aparece como `device` no `adb`.
- [ ] Projeto abre no emulador a partir da pasta correta.
- [ ] Alterações de interface aparecem durante o desenvolvimento.
- [ ] Login, navegação e persistência testados conforme o aplicativo.
- [ ] Diferença entre desenvolvimento com Metro e APK de release compreendida.

## Referências

- [Expo — Android Studio Emulator](https://docs.expo.dev/workflow/android-studio-emulator/)
- [Expo — CLI e atalhos](https://docs.expo.dev/more/expo-cli/)
- [Expo — build de desenvolvimento local](https://docs.expo.dev/guides/local-app-development/)
- [Android — criar e gerenciar dispositivos virtuais](https://developer.android.com/studio/run/managing-avds)
- [Android — aceleração do emulador](https://developer.android.com/studio/run/emulator-acceleration)
- [Android — rede do emulador](https://developer.android.com/studio/run/emulator-networking)
