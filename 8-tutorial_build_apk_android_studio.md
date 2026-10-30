# Tutorial 8 — Android Studio e geração de APK no Windows 11
### ARA0089 — Programação para Dispositivos Móveis em Android

Neste roteiro vamos instalar e configurar o **Android Studio**, preparar um projeto **React Native com Expo** e gerar um **APK de release assinado** no próprio computador. Este roteiro usa exclusivamente **Windows 11**.

Escolha o roteiro do seu sistema: [Windows 11](8-tutorial_build_apk_android_studio.md), [macOS](8.1-tutorial_build_apk_android_studio_macos.md) ou [Linux](8.2-tutorial_build_apk_android_studio_linux.md). Cada arquivo contém o fluxo completo de instalação e geração do APK.

Vamos continuar o **MeuAppFirebase**, dos tutoriais [6](6-tutorial_autenticacao_firebase_email_senha.md), [6.1](6.1-tutorial_cadastro_usuarios_firebase.md) e [6.2](6.2-tutorial_crud_produtos_firestore.md). O procedimento também serve para os outros aplicativos Expo da disciplina.

**Ao final, você conseguirá:**

- instalar Android Studio, Android SDK e ferramentas de compilação;
- configurar os caminhos do SDK e do Java;
- gerar o projeto Android a partir do Expo e abri-lo na IDE;
- diferenciar um APK de debug de um APK de release;
- criar uma chave de assinatura, gerar um APK e instalá-lo;
- gerar atualizações e identificar falhas comuns.

> **Onde executar os comandos?** Na pasta do aplicativo criado nas aulas, onde está o `package.json`. Este repositório contém somente os roteiros; não execute os comandos de build na pasta dos tutoriais.

---

## 1. Entender o processo

No [tutorial 7](7-tutorial_build_apk_android_eas.md), usamos EAS Build na nuvem. Aqui, a compilação será local, com Android Studio e Gradle. Não é necessário criar conta Expo ou conta de desenvolvedor Google Play para gerar e instalar este APK de teste.

```text
Projeto Expo (JavaScript, imagens e configurações)
                       ↓ npx expo prebuild --platform android
Pasta android/ (projeto nativo)
                       ↓ Android Studio + Gradle + assinatura
APK de release
                       ↓ instalação
Aplicativo no celular, sem Expo Go e sem Metro
```

| Resultado | Uso |
|---|---|
| APK de debug | Depurar; no fluxo normal de React Native/Expo, depende do Metro para carregar o JavaScript |
| APK de release assinado | Instalar e testar o aplicativo com JavaScript e recursos incorporados |
| AAB | Formato para distribuição pela Google Play; não é instalado diretamente ao tocar no arquivo |

O APK de release abre sem o computador. Login e consultas ao Firebase continuam precisando de internet. Compilar localmente exige downloads de SDK, dependências e ferramentas, especialmente na primeira execução. Consulte também o [guia de release local do Expo](https://docs.expo.dev/guides/local-app-production/).

## 2. Preparar o computador com Windows 11

### 2.1. Organizar a pasta de trabalho

Use um caminho curto, por exemplo:

```text
C:\Projetos\MeuAppFirebase
```

Evite pastas sincronizadas pelo OneDrive e caminhos muito longos. Isso reduz conflitos de sincronização e problemas com ferramentas nativas.

Tenha espaço livre para SDKs, caches e arquivos de compilação. Confira os requisitos vigentes na [página de instalação do Android Studio](https://developer.android.com/studio/install). Se o emulador ficar pesado, use um celular físico.

### 2.2. Instalar Node.js e Git

1. Instale uma versão **LTS do Node.js compatível com o Expo SDK do seu projeto**, pelo [site do Node.js](https://nodejs.org/).
2. Instale o Git pelo [site oficial](https://git-scm.com/downloads).
3. Feche e reabra o terminal.
4. No **PowerShell**, confira:

```powershell
node --version
npm --version
git --version
```

Se o PowerShell bloquear `npm.ps1` ou `npx.ps1`, use `npm.cmd` e `npx.cmd` nos comandos deste roteiro, ou execute os comandos no Prompt de Comando. Não é necessário mudar a política de segurança do Windows para seguir a aula.

### 2.3. Instalar o Android Studio

1. Baixe a versão **estável**, adequada ao seu computador, em [developer.android.com/studio](https://developer.android.com/studio).
2. Execute o instalador e mantenha o Android Studio selecionado.
3. Abra a IDE e conclua o **Setup Wizard**.
4. Aceite as licenças dos componentes necessários e aguarde os downloads.
5. Na tela inicial, abra **More Actions > SDK Manager**. Com um projeto aberto, procure **Tools > SDK Manager**.

Anote o campo **Android SDK Location**. Um caminho comum é:

```text
C:\Users\SEU_USUARIO\AppData\Local\Android\Sdk
```

Use o caminho mostrado no seu computador, mesmo que seja diferente. Os nomes dos menus podem variar entre versões da IDE.

### 2.4. Instalar os componentes do SDK

No **SDK Manager**:

| Aba | Componentes |
|---|---|
| SDK Platforms | Plataforma Android exigida pelo `compileSdk` do projeto |
| SDK Tools | Android SDK Platform-Tools, Android SDK Build-Tools e Android SDK Command-line Tools (latest) |
| SDK Tools, se usar emulador | Android Emulator |
| SDK Tools, se solicitado pelo build | NDK (Side by side) e CMake nas versões pedidas |

Clique em **Apply**, aceite as licenças e espere terminar. Depois do prebuild, a sincronização do projeto indicará componentes adicionais necessários. Use **Show Package Details** para localizar versões específicas.

**Não escolha um número de API arbitrário:** a versão do Expo/React Native determina os requisitos nativos. Confira `compileSdk`, `buildToolsVersion` e `ndkVersion` na configuração gerada, como `android/build.gradle` e propriedades relacionadas. Se os valores forem indiretos, use a versão informada no erro de sincronização para instalar o componente correspondente.

### 2.5. Configurar o SDK no Windows

Pesquise no menu Iniciar por **Editar as variáveis de ambiente da sua conta**. Nas variáveis do usuário:

1. Crie `ANDROID_HOME` com o caminho do SDK anotado anteriormente.
2. Edite `Path` e adicione estas entradas separadamente:

```text
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\emulator
%ANDROID_HOME%\cmdline-tools\latest\bin
```

A entrada `emulator` é necessária somente se você usar o emulador. Não substitua o conteúdo existente do `Path`. A variável `ANDROID_SDK_ROOT` está obsoleta; se já existir, evite deixá-la apontando para outro SDK. Referência: [variáveis das ferramentas Android](https://developer.android.com/tools/variables).

Feche e reabra o PowerShell:

```powershell
$env:ANDROID_HOME
adb --version
sdkmanager.bat --licenses
```

Leia e aceite as licenças necessárias quando solicitado. Se `sdkmanager.bat` não for encontrado, confira a instalação de **Command-line Tools (latest)** e o `Path`.

### 2.6. Instalar e configurar Java 17 para o build/deploy

**Use o JDK 17 para compilar e instalar o app desta aula.** Essa é a versão recomendada na [configuração de ambiente do React Native](https://reactnative.dev/docs/set-up-your-environment), para evitar incompatibilidades com versões mais novas do Java. Instalar o Android Studio não garante que o Gradle usará Java 17. Se você atualizar Expo, React Native ou AGP no futuro, confira novamente os requisitos antes de manter essa configuração.

No PowerShell, instale o **JDK**, não somente o JRE:

```powershell
winget install --id EclipseAdoptium.Temurin.17.JDK --exact
```

Se não tiver `winget`, instale Temurin **17 JDK** pelo [site do Adoptium](https://adoptium.net/installation). Depois localize a pasta instalada:

```powershell
Get-ChildItem 'C:\Program Files\Eclipse Adoptium' -Directory
```

Copie o caminho exato da pasta `jdk-17...` e substitua o exemplo abaixo. Não inclua `bin` no `JAVA_HOME`:

```powershell
$env:JAVA_HOME = 'C:\Program Files\Eclipse Adoptium\jdk-17.SEUA_VERSAO-hotspot'
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
java -version
javac -version
```

Os dois comandos devem mostrar **17.x**. Para persistir o diretório validado na conta do usuário:

```powershell
[Environment]::SetEnvironmentVariable('JAVA_HOME', $env:JAVA_HOME, 'User')
```

Em **Editar as variáveis de ambiente da sua conta > Path**, adicione `%JAVA_HOME%\bin` antes de outras entradas Java. Reabra os terminais e confira `java -version`. Isso evita sobrescrever todo o `Path` com um comando.

Na IDE, abra **Settings > Build, Execution, Deployment > Build Tools > Gradle**. Em **Gradle JDK**, escolha **JAVA_HOME** ou adicione a pasta do **JDK 17** instalada. Não selecione automaticamente o `jbr` do Android Studio: ele pode ser de outra versão. Reinicie a IDE após mudar variáveis de ambiente.

O Android Studio pode continuar usando seu runtime próprio; estamos selecionando Java 17 para o **Gradle do aplicativo**. A IDE e o terminal têm seleções independentes, conforme a [documentação de JDKs Android](https://developer.android.com/build/jdks).

Se `sdkmanager.bat` exigir Java mais novo, use o runtime do Android Studio somente em um terminal separado para instalar o SDK. Volte ao terminal configurado com Java 17 antes do build.

## 3. Preparar o aplicativo Expo

No PowerShell, entre no seu projeto:

```powershell
cd C:\Projetos\MeuAppFirebase
```

Se acabou de baixar o projeto, execute **uma** das opções:

- `npm ci`, se houver `package-lock.json` consistente;
- `npm install`, se não houver lockfile.

Depois, verifique as dependências:

```powershell
npx expo-doctor
npx expo install --check
```

Corrija os problemas apresentados. Se necessário, use `npx expo install --fix`, revise as mudanças e teste o app:

```powershell
npx expo start --clear
```

Confira login, cadastro, navegação e CRUD. Pare o servidor com `Ctrl+C` antes de prosseguir.

### 3.1. Identificação e versão

No `app.json`, ajuste os seguintes campos **preservando as outras configurações**:

```json
{
  "expo": {
    "name": "Meu App Firebase",
    "slug": "meu-app-firebase",
    "version": "1.0.0",
    "android": {
      "package": "br.com.seunome.meuappfirebase",
      "versionCode": 1
    }
  }
}
```

Esse trecho é um exemplo parcial. Mescle os campos no objeto `expo` e no objeto `android` existentes; mantenha imagens, plugins e demais opções. Se usar `app.config.js` ou `app.config.ts`, altere a configuração correspondente.

Substitua `seunome` por um identificador próprio, sem espaços, acentos ou hífens. O pacote identifica o app no Android e deve ser mantido nas atualizações.

### 3.2. Variáveis do Firebase

Se o app usa Firebase, mantenha na **raiz do aplicativo** o `.env.local` configurado no tutorial 6, com os valores reais do seu projeto:

```dotenv
EXPO_PUBLIC_FIREBASE_API_KEY=SEU_VALOR
EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN=SEU_VALOR
EXPO_PUBLIC_FIREBASE_PROJECT_ID=SEU_VALOR
EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET=SEU_VALOR
EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=SEU_VALOR
EXPO_PUBLIC_FIREBASE_APP_ID=SEU_VALOR
```

Não copie `SEU_VALOR` literalmente. No código, use acesso estático, como `process.env.EXPO_PUBLIC_FIREBASE_API_KEY`. O empacotamento com Expo CLI carrega essas variáveis locais; valores cadastrados somente no EAS não são baixados automaticamente pelo Android Studio.

Variáveis `EXPO_PUBLIC_` ficam incorporadas ao app e podem ser extraídas. Nunca coloque nelas senhas administrativas ou chaves privadas. A proteção dos dados depende da autenticação e das regras do Firebase. Mantenha `.env.local` fora do Git. Referência: [variáveis de ambiente do Expo](https://docs.expo.dev/guides/environment-variables/).

O aplicativo das aulas usa o **SDK JavaScript do Firebase**, cadastrado como aplicativo Web. Ele não exige `google-services.json` somente por ser compilado para Android. Bibliotecas nativas de Firebase podem exigir configuração adicional própria.

## 4. Gerar a pasta Android

Salve seu trabalho e faça um commit ou backup antes de gerar arquivos nativos. Na raiz do app:

```powershell
npx expo prebuild --platform android
```

O comando gera a pasta `android/` usando a configuração Expo e os plugins. Se solicitado, confirme o identificador Android definido no passo anterior. Confira o resultado:

```text
MeuAppFirebase/
├── App.js
├── app.json
├── package.json
├── .env.local
└── android/
    ├── app/
    │   └── build.gradle
    ├── build.gradle
    ├── gradle.properties
    ├── gradlew
    ├── gradlew.bat
    └── settings.gradle
```

A estrutura pode variar com a versão do SDK. Projetos que já possuem `android/` com alterações manuais precisam de revisão antes de reaplicar o prebuild.

> **Cuidado com `--clean`:** essa opção apaga e recria a pasta nativa, podendo remover alterações manuais. Não a use como tentativa automática de corrigir erros. Configurações permanentes devem ser representadas no app config/config plugins quando possível. Veja [Continuous Native Generation no Expo](https://docs.expo.dev/workflow/continuous-native-generation/).

## 5. Abrir e sincronizar no Android Studio

1. Abra o Android Studio e clique em **Open**.
2. Selecione **`C:\Projetos\MeuAppFirebase\android`**, a pasta que contém `settings.gradle`.
3. Confirme a confiança no projeto se você reconhece sua origem.
4. Aguarde o **Gradle Sync** e os downloads.
5. Instale componentes faltantes indicados pela IDE e aceite as licenças.
6. Abra **Build** ou **Sync** para consultar erros, se houver.

Não crie um projeto Android vazio pelo assistente: vamos abrir o projeto gerado pelo Expo. Não aceite migrações de AGP/Gradle sugeridas pela IDE sem conferir a compatibilidade com seu Expo SDK; utilize inicialmente as versões geradas.

### Se aparecer “SDK location not found”

Confira o SDK Manager. Se necessário, crie ou ajuste `android/local.properties` com o caminho real do SDK, usando barras `/` no Windows:

```properties
sdk.dir=C:/Users/SEU_USUARIO/AppData/Local/Android/Sdk
```

Esse arquivo é específico do computador e deve ficar fora do Git.

### Se aparecer “Cannot run program node”

Confirme `node --version` em um terminal novo. Feche e abra novamente a IDE após instalar Node.js ou alterar o `Path`. A compilação React Native executa o Node para empacotar o JavaScript; apenas instalar o Android Studio não basta.

## 6. Teste opcional de desenvolvimento

Você pode usar um celular com **Opções do desenvolvedor > Depuração USB** habilitada. Conecte um cabo de dados, aceite a autorização no celular e confira:

```powershell
adb devices
```

O aparelho deve aparecer como `device`. Se estiver `unauthorized`, desbloqueie-o e aceite a autorização. Alguns fabricantes exigem driver USB no Windows.

Para usar emulador, abra **Tools > Device Manager**, crie um dispositivo, instale uma imagem compatível e inicie-o. No Windows 11, a aceleração pode exigir virtualização habilitada no BIOS/UEFI e o recurso **Windows Hypervisor Platform**, seguido de reinicialização. Veja a [configuração oficial de aceleração](https://developer.android.com/studio/run/emulator-acceleration).

Para testar em debug, selecione o dispositivo e use **Run** na IDE. Execute `npx expo start` na raiz do app para disponibilizar o Metro. Em um aparelho USB, se necessário, use `adb reverse tcp:8081 tcp:8081`.

Esse teste é opcional: você pode seguir diretamente para o APK de release e testá-lo no celular.

## 7. Gerar um APK de debug, se precisar

No Android Studio, selecione a variante `debug` em **Build Variants** e procure **Build > Generate Bundle(s) / APK(s) > Generate APK(s)**. Em outras versões, a opção aparece como **Build Bundle(s) / APK(s) > Build APK(s)**.

O arquivo normalmente fica em:

```text
android/app/build/outputs/apk/debug/app-debug.apk
```

Use a notificação **Locate** para localizar o resultado. O botão **Run** instala uma versão de desenvolvimento; para obter um arquivo compartilhável, use o comando de geração de APK. Veja [build e execução no Android Studio](https://developer.android.com/studio/run).

**Para entregar um aplicativo que abre sem Metro, continue com a versão de release.**

## 8. Criar a chave e gerar o APK de release assinado

1. Abra **Build > Generate Signed Bundle / APK**.
2. Selecione **APK** e clique em **Next**.
3. Se solicitado, selecione o módulo **app**.
4. No primeiro build, clique em **Create new** para criar o keystore.
5. Preencha:

| Campo | Exemplo/orientação |
|---|---|
| Key store path | `C:\ChavesAndroid\meuappfirebase-release.jks`, fora do repositório |
| Password | Senha forte, guardada em local seguro |
| Alias | `meuappfirebase` |
| Key password | Use a mesma senha do keystore para compatibilidade |
| Validity | Pelo menos 25 anos |
| Certificate | Preencha os dados de identificação solicitados |

6. Confirme e retorne ao assistente.
7. Confira caminho, alias e senhas; avance.
8. Escolha **release**, confira a pasta de destino e conclua em **Create/Finish**.

Se já existe uma chave desse aplicativo, use **Choose existing**. Guarde o `.jks`, alias e senhas com backup seguro. Mantenha chaves e senhas fora do Git. Atualizações por APK exigem a mesma assinatura. Referência: [assinatura de aplicativos Android](https://developer.android.com/studio/publish/app-signing).

### 8.1. Encontrar o resultado

Aguarde a conclusão sem erros. Clique em **Locate** na notificação e confira o destino escolhido no assistente. Dependendo da configuração, o APK pode aparecer em:

```text
android/app/release/app-release.apk
```

ou:

```text
android/app/build/outputs/apk/release/app-release.apk
```

O destino exibido pela IDE é a referência. O arquivo de entrega deve ser um **APK de release assinado**, não um `app-release-unsigned.apk` ou um AAB.

O build de release do projeto Expo gerado empacota JavaScript e recursos. Se você personalizou configurações Gradle para pular o bundle, revise-as antes de usar o APK como entrega independente.

## 9. Instalar e testar o APK

### Opção A — Transferir para o celular

1. Copie o APK por USB ou outro meio de sua preferência.
2. Abra o arquivo no Android.
3. Se solicitado, permita **Instalar apps desconhecidos** somente para o aplicativo usado para abrir o arquivo.
4. Instale e abra o app pelo próprio ícone.

### Opção B — Instalar pelo computador

Com depuração USB autorizada, execute no PowerShell, substituindo o caminho pelo APK gerado:

```powershell
adb devices
adb install -r "C:\Projetos\MeuAppFirebase\android\app\release\app-release.apk"
```

O `-r` solicita reinstalação preservando dados, desde que o pacote e a assinatura sejam compatíveis.

### Teste de entrega

- Pare o Metro e desconecte o cabo USB.
- Abra o app pelo ícone e confira que não pede servidor de desenvolvimento.
- Teste login, cadastro, navegação e CRUD com internet.
- Se usar SQLite, salve dados, feche e reabra o app.
- Teste sem internet para observar o comportamento das telas que dependem da rede.

## 10. Gerar uma atualização

1. Altere o código e teste as funcionalidades.
2. No app config, aumente `version`, por exemplo para `1.0.1`, e `android.versionCode`, por exemplo para `2`.
3. Se mantém o projeto nativo gerado pelo Expo, reaplique `npx expo prebuild --platform android`, após salvar e revisar alterações nativas.
4. Confira em `android/app/build.gradle` se `applicationId`, `versionName` e `versionCode` refletem os valores esperados.
5. Sincronize na IDE e gere outro release usando **o mesmo keystore e alias**.
6. Instale por cima e verifique os dados existentes.

Se gerencia o Android manualmente, atualize os campos de versão no projeto nativo; modificar somente `app.json` não atualiza automaticamente uma pasta Android já gerada.

Um APK anterior do EAS pode ter outra assinatura. Criar uma chave local nova não permite atualizar esse APK. Para continuidade, use a mesma chave com que ele foi assinado. Desinstalar para trocar a assinatura remove os dados locais do app; preserve o que precisar antes de optar por isso.

## 11. Comandos Gradle opcionais

O fluxo principal de assinatura deste roteiro usa a IDE. Para diagnóstico ou debug, você também pode executar o Gradle Wrapper fornecido pelo projeto.

**Windows, no PowerShell:**

```powershell
cd android
.\gradlew.bat --version
.\gradlew.bat assembleDebug
cd ..
```

Para um build de release, a tarefa é `assembleRelease`, mas **a tarefa sozinha não configura sua chave de distribuição**. Alguns templates usam a chave de debug até na configuração release. O assistente de assinatura da seção 8 seleciona explicitamente o keystore; para automatizar no terminal, configure e verifique a assinatura release no Gradle sem colocar senhas no repositório.

### 11.1. Selecionar Java 17 e fazer deploy de desenvolvimento

Na raiz do aplicativo, **depois de gerar `android/`**, execute o bloco abaixo. A variável persistente precisa ter sido configurada na seção 2.6.

```powershell
$env:JAVA_HOME = [Environment]::GetEnvironmentVariable('JAVA_HOME', 'User')
$env:Path = "$env:JAVA_HOME\bin;$env:Path"
java -version
javac -version
cd android
.\gradlew.bat --stop
.\gradlew.bat --version
.\gradlew.bat installDebug
cd ..
```

Confira **17** tanto no Java quanto na JVM usada pelo Gradle. `--stop` encerra daemons anteriores; `installDebug` compila e instala no emulador ou celular conectado. Deixe somente o dispositivo desejado conectado. O debug normalmente precisa de Metro: execute `npx expo start` na raiz em outro terminal (`--dev-client` se usar `expo-dev-client`).

Se o Gradle ainda usar outra JVM, revise **Gradle JDK**, `org.gradle.java.home` em propriedades locais/do usuário e, se existir, `android/gradle/gradle-daemon-jvm.properties`. Critérios do daemon podem prevalecer sobre `JAVA_HOME`; confira a versão efetiva antes de compilar. Não grave caminhos pessoais ou senhas no Git. Veja os [critérios da JVM do Gradle](https://docs.gradle.org/current/userguide/gradle_daemon.html#sec:daemon_jvm_criteria).

### 11.2. Gerar o release com Java 17

Com Java 17 selecionado, use o assistente da **seção 8** para gerar o APK assinado. Para automatizar pelo terminal, configure primeiro a assinatura release no Gradle, com sua chave e senhas fora do Git. Só depois execute, na pasta `android/`:

```powershell
.\gradlew.bat assembleRelease
```

O APK costuma ficar em `android/app/build/outputs/apk/release/`. Instale-o com `adb install -r` conforme a seção 9. Java 17 é uma configuração de compilação do computador; não precisa ser instalado no celular.

## 12. Problemas comuns

| Problema | O que verificar |
|---|---|
| `adb` não reconhecido | Instalação de Platform-Tools, `Path` e reabertura do terminal |
| `SDK location not found` | `ANDROID_HOME` e `android/local.properties` |
| Licenças não aceitas | Execute `sdkmanager.bat --licenses` |
| SDK, NDK ou CMake ausente | Instale a versão indicada no erro pelo SDK Manager |
| Java/Gradle incompatível | Compare Gradle JDK da IDE, `JAVA_HOME` e saída de `gradlew --version` |
| Download de dependências falha | Confira internet, proxy e modo offline do Gradle; leia o erro de download |
| Falha em tarefa CMake ou caminho longo | Use pasta curta, como `C:\Projetos\MeuAppFirebase`, e confira NDK/CMake |
| `Cannot run program node` | Node no `Path` acessível à IDE; reinicie-a após a instalação |
| APK pede Metro | Confira se instalou o release e se o bundle JavaScript foi gerado |
| Firebase funciona no Expo Go, mas não no APK | Confira `.env.local` na raiz, acesso estático às variáveis e gere novo release |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | App existente assinado com outra chave; use a assinatura original |
| `INSTALL_FAILED_VERSION_DOWNGRADE` | Gere versão com `versionCode` superior |
| `INSTALL_FAILED_NO_MATCHING_ABIS` | Confira arquitetura do aparelho e arquiteturas incluídas no build |
| App fecha ao abrir | Consulte **Logcat**, selecione o processo e procure a exceção correspondente |

Não altere versões de Gradle, Java, React Native e Expo simultaneamente para tentar resolver uma falha. Leia a primeira causa relevante no painel Build e corrija o requisito indicado.

## 13. Checklist e atividade

- [ ] Android Studio e componentes do SDK instalados.
- [ ] Node e SDK acessíveis; `java`, `javac` e a JVM do Gradle conferidos na versão 17.
- [ ] Projeto verificado com Expo Doctor.
- [ ] Pacote Android e versões definidos.
- [ ] Firebase configurado localmente, quando utilizado.
- [ ] Pasta `android/` gerada e sincronizada na IDE.
- [ ] Keystore criado ou selecionado, com backup seguro.
- [ ] APK de release assinado localizado e instalado.
- [ ] App abre com Metro parado e sem Expo Go.
- [ ] Login, CRUD e persistência testados conforme o projeto.
- [ ] Chaves, senhas, `.env.local` e `local.properties` fora do Git.

**Atividade:** entregue pelo canal indicado pelo professor o APK assinado, o link do repositório do aplicativo e uma captura da versão instalada. Gere depois uma atualização, mantendo pacote e assinatura, e registre se os dados locais foram preservados. Não entregue o keystore nem suas senhas.

## Referências

- [Android Studio — download](https://developer.android.com/studio)
- [Android Studio — instalação por sistema operacional](https://developer.android.com/studio/install)
- [Android — variáveis de ambiente](https://developer.android.com/tools/variables)
- [Android — versões Java nos builds](https://developer.android.com/build/jdks)
- [Android Studio — build e execução](https://developer.android.com/studio/run)
- [Android Studio — assinatura](https://developer.android.com/studio/publish/app-signing)
- [Android Emulator — aceleração](https://developer.android.com/studio/run/emulator-acceleration)
- [Expo — geração do projeto nativo](https://docs.expo.dev/workflow/continuous-native-generation/)
- [Expo — release local](https://docs.expo.dev/guides/local-app-production/)
- [Expo — variáveis de ambiente](https://docs.expo.dev/guides/environment-variables/)
