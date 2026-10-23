# Tutorial 7 — Build do aplicativo e geração de APK para Android
### ARA0089 — Programação para Dispositivos Móveis em Android

Até aqui, executamos nossos aplicativos usando o **Expo Go**. Nesta aula vamos transformar um projeto React Native com Expo em um **aplicativo instalável no Android**, com ícone próprio e sem depender do computador para abrir.

**Ao final deste tutorial você saberá:**

- explicar o que é um build e distinguir APK de AAB;
- criar uma conta Expo e configurar o **EAS Build**;
- gerar um APK na nuvem e instalar em um celular Android;
- preparar as variáveis do Firebase para o build;
- testar o app instalado e gerar uma nova versão;
- reconhecer as alternativas para compilar no próprio computador.

Vamos usar o **MeuAppFirebase**, desenvolvido nos [tutoriais 6](6-tutorial_autenticacao_firebase_email_senha.md), [6.1](6.1-tutorial_cadastro_usuarios_firebase.md) e [6.2](6.2-tutorial_crud_produtos_firestore.md). O processo também se aplica aos projetos Expo anteriores; se o seu app não usa Firebase, pule a seção de variáveis de ambiente.

> **Onde executar os comandos?** Na pasta do aplicativo, onde está o `package.json`. Este repositório contém os roteiros das aulas; os comandos de build devem ser executados no projeto que você criou ao acompanhá-las.

---

## 1. Entendendo o que vamos gerar

### 1.1 O que significa fazer um build?

Durante o desenvolvimento, o Expo Go carrega nosso código por meio do servidor iniciado com `npx expo start`. Para entregar o aplicativo, precisamos reunir o código JavaScript, as imagens e os componentes nativos em um pacote Android. Esse processo é chamado de **build** ou **compilação**.

```text
Código React Native + imagens + bibliotecas + configurações
                            ↓
                     Processo de build
                            ↓
                      Arquivo .apk
                            ↓
               Instalação no celular Android
```

O APK deste roteiro incorpora o código e os recursos do aplicativo. Ele abre sem Expo Go e sem o Metro rodando no computador. Recursos que consultam serviços externos, como login e operações no Firebase, continuam precisando de conexão.

### 1.2 APK, AAB e Expo Go

| Termo | O que é | Quando usar |
|---|---|---|
| **Expo Go** | Aplicativo que carrega projetos compatíveis durante o desenvolvimento | Programar e testar rapidamente |
| **APK** (`.apk`) | Pacote de instalação Android | Instalar diretamente no celular e compartilhar para testes |
| **AAB** (`.aab`) | Pacote de publicação que permite à Google Play gerar APKs adequados aos aparelhos | Distribuir pela Google Play |

O AAB não é instalado diretamente tocando no arquivo no celular. Para esta aula, vamos solicitar explicitamente **APK**, conforme a [documentação de builds Android do Expo](https://docs.expo.dev/build-reference/apk/).

### 1.3 Desenvolvimento, preview e produção

- **Development build:** aplicativo preparado para desenvolver e depurar, normalmente conectado ao Metro e com `expo-dev-client`.
- **Preview:** nome que daremos ao perfil para gerar um APK de teste que abre sozinho.
- **Production:** perfil que usaremos para gerar AAB, caso futuramente seja necessário publicar na loja.

Os nomes dos perfis são convenções. As opções dentro de cada perfil determinam o resultado. Neste tutorial, o perfil `preview` terá `developmentClient: false`.

---

## 2. Escolhendo onde fazer o build

O **EAS** significa **Expo Application Services**. Um de seus serviços é o **EAS Build**, que compila o aplicativo em servidores do Expo.

| Caminho | Conta Expo | Ambiente Android no computador | Indicação |
|---|---|---|---|
| EAS Build na nuvem | Sim | Não é necessário | Caminho principal desta aula, inclusive no Windows |
| EAS Build com `--local` | Sim | Necessário | Quem já tem ambiente nativo configurado em macOS/Linux |
| Build nativo com Gradle/Android Studio | Não exige EAS | Necessário | Quem precisa controlar o projeto Android e a assinatura |

A conta Expo é diferente da conta Google/Firebase e da conta de desenvolvedor Google Play. **Você não precisa de conta Google Play para gerar e instalar o APK desta aula.**

O EAS Build está disponível também no plano gratuito, sujeito a limites e filas. Consulte as condições vigentes na [página de planos do Expo](https://expo.dev/pricing); tempo de espera e cotas podem mudar. A [configuração oficial do EAS Build](https://docs.expo.dev/build/setup/) apresenta os requisitos do serviço.

---

## 3. Preparar e verificar o projeto

No terminal, entre na pasta do aplicativo:

```bash
cd MeuAppFirebase
```

Se você acabou de baixar o projeto, instale as dependências com `npm ci` quando houver um `package-lock.json` consistente. Se não houver esse arquivo, use `npm install`.

Verifique o ambiente e as dependências:

```bash
node -v
npm -v
npx expo-doctor
npx expo install --check
```

Leia os resultados e corrija os problemas antes de compilar. Caso o Expo aponte versões incompatíveis, `npx expo install --fix` pode ajustar as dependências; revise as alterações e teste novamente.

Agora execute:

```bash
npx expo start --clear
```

Teste login, cadastro de usuários, navegação e CRUD de produtos. Se o seu projeto usa SQLite, teste também salvar, fechar e reabrir o app.

> **Verificação:** o app precisa funcionar no ambiente de desenvolvimento antes de você tentar gerar o instalador. O build não corrige erros de código ou regras incorretas do Firebase.

---

## 4. Configurar a identificação do aplicativo

Abra o `app.json` na raiz do projeto e ajuste os campos abaixo **dentro do objeto `expo` que já existe**:

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

> O trecho mostra os campos relevantes, não o arquivo completo do seu projeto. Preserve `icon`, `splash`, `plugins`, `android.adaptiveIcon` e as demais configurações existentes. Se já houver um objeto `android`, acrescente os campos nele. Em projetos com `app.config.js` ou `app.config.ts`, faça os ajustes na configuração correspondente.

| Campo | Significado |
|---|---|
| `name` | Nome exibido no celular |
| `slug` | Nome usado para identificar o projeto no Expo, junto da conta proprietária |
| `version` | Versão legível do aplicativo, como `1.0.0` |
| `android.package` | Identificador Android do aplicativo |
| `android.versionCode` | Número inteiro usado para ordenar versões Android |

Substitua `seunome` por um identificador seu, sem espaços, acentos ou hífens. Por exemplo: `br.com.maria.meuappfirebase`. Use um identificador próprio e mantenha-o nas próximas versões: mudar o pacote faz o Android tratar o app como outro aplicativo.

Confira também os caminhos das imagens já configuradas. Um ícone que existe no computador, mas foi excluído do envio ao EAS, pode causar falha no build.

---

## 5. Criar a conta e conectar o projeto ao EAS

### 5.1 Criar uma conta Expo

1. Acesse [expo.dev/signup](https://expo.dev/signup).
2. Faça seu cadastro e siga as instruções de confirmação exibidas.
3. Entre no painel com a conta criada.

### 5.2 Acessar a CLI do EAS

Vamos usar `npx eas-cli@latest` para executar a ferramenta sem instalação global:

```bash
npx eas-cli@latest --version
npx eas-cli@latest login
npx eas-cli@latest whoami
```

Faça login quando solicitado. O último comando deve exibir seu usuário Expo.

> A CLI do **Expo** (`npx expo ...`) e a CLI do **EAS** (`npx eas-cli@latest ...`) têm funções diferentes. Aqui, a primeira ajuda no desenvolvimento e a segunda configura e solicita builds.

### 5.3 Configurar o projeto

Dentro da pasta do app, execute:

```bash
npx eas-cli@latest build:configure
```

Se solicitado:

1. confirme a criação/vinculação do projeto na conta correta;
2. selecione **Android** como plataforma;
3. confira o identificador Android definido no passo anterior.

A ferramenta cria o `eas.json` e pode adicionar informações de vínculo ao Expo, como `extra.eas.projectId`, na configuração do app. **Preserve os valores gerados**; não copie o identificador do projeto de outro colega.

> **Verificação:** abra o painel do Expo e confirme que o projeto aparece na sua conta. O GitHub não é obrigatório para solicitar esse build pelo terminal.

---

## 6. Configurar o perfil que gera APK

Abra o `eas.json`. Para um projeto novo nesta configuração, use:

```json
{
  "build": {
    "preview": {
      "distribution": "internal",
      "developmentClient": false,
      "environment": "preview",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "environment": "production",
      "android": {
        "buildType": "app-bundle"
      }
    }
  }
}
```

Se você já usa EAS, mantenha configurações necessárias existentes e acrescente/ajuste esses perfis dentro de `build`.

- `distribution: "internal"` prepara a distribuição para testes fora da loja.
- `developmentClient: false` evita gerar um cliente de desenvolvimento neste perfil.
- `environment: "preview"` escolhe o conjunto de variáveis de ambiente usado no build.
- `android.buildType: "apk"` solicita o formato que instalaremos diretamente.
- `production` foi incluído para demonstrar a diferença: esse perfil solicita um AAB.

Veja as opções na [referência do eas.json](https://docs.expo.dev/eas/json/).

> **Atenção:** neste roteiro o comando de build deve incluir `--profile preview`. Escolher `production` gera outro tipo de arquivo.

---

## 7. Preparar as variáveis do Firebase para a nuvem

Nos tutoriais anteriores, criamos um `.env.local` ignorado pelo Git. Esse arquivo funciona no seu computador, mas **não é enviado normalmente ao build remoto**. Por isso, vamos cadastrar os valores no ambiente do projeto no EAS.

### 7.1 Cadastrar no painel

1. Abra seu projeto no painel do Expo.
2. Entre em **Project settings → Environment variables**.
3. Use a opção de adicionar variáveis.
4. Cadastre as seis variáveis abaixo com os valores reais do seu `.env.local`.
5. Selecione o ambiente **preview**, igual ao definido no `eas.json`.
6. Para essas configurações públicas do cliente Firebase, use a visibilidade **Plain text**.

| Nome da variável | Valor a copiar do Firebase |
|---|---|
| `EXPO_PUBLIC_FIREBASE_API_KEY` | `apiKey` |
| `EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN` | `authDomain` |
| `EXPO_PUBLIC_FIREBASE_PROJECT_ID` | `projectId` |
| `EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET` | `storageBucket` |
| `EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` | `messagingSenderId` |
| `EXPO_PUBLIC_FIREBASE_APP_ID` | `appId` |

Confira o ambiente pelo terminal:

```bash
npx eas-cli@latest env:list --environment preview
```

O passo a passo do painel está na [documentação de gerenciamento de variáveis do EAS](https://docs.expo.dev/eas/environment-variables/manage/). O campo `environment` do perfil determina quais variáveis serão usadas, conforme a [documentação de uso em builds](https://docs.expo.dev/eas/environment-variables/usage/).

### 7.2 Por que não basta esconder no `.env.local`?

O prefixo `EXPO_PUBLIC_` faz os valores serem incorporados ao código entregue no aplicativo. Manter o arquivo fora do Git evita versionar configurações locais, mas **não transforma esses valores em segredos dentro do APK**.

A configuração do cliente Firebase identifica o projeto; a proteção dos dados depende da autenticação e das **regras de segurança do Firestore**. Nunca coloque senha de usuário, chave privada de conta de serviço ou credenciais do Firebase Admin SDK em `EXPO_PUBLIC_`. Veja a [explicação do Expo sobre variáveis públicas](https://docs.expo.dev/guides/environment-variables/).

> Para o Firebase JavaScript SDK usado nas aulas, mantenha as configurações do aplicativo **Web** criado no Firebase. Não é necessário migrar para um SDK nativo ou adicionar `google-services.json` apenas para gerar este APK.

---

## 8. Solicitar o build do APK

Execute na raiz do app:

```bash
npx eas-cli@latest build --platform android --profile preview
```

### 8.1 Entender a assinatura Android

Na primeira compilação, o EAS poderá perguntar se deseja gerar uma **Android Keystore**. Para um aplicativo novo desta aula, aceite a geração e o gerenciamento pelo EAS.

A keystore contém a chave usada para **assinar** o aplicativo. O Android usa essa assinatura para verificar se uma atualização vem do mesmo responsável. Para atualizar o mesmo app, preserve o pacote e a chave de assinatura.

Se o app já foi distribuído anteriormente, use as credenciais correspondentes em vez de gerar uma chave nova. Você pode consultar o gerenciamento de credenciais com:

```bash
npx eas-cli@latest credentials --platform android
```

Guarde backups da chave e das senhas em local seguro quando baixá-los; não os inclua no Git. Veja a [documentação de assinatura de aplicativos](https://docs.expo.dev/app-signing/app-credentials/).

### 8.2 Acompanhar a compilação

A CLI prepara e envia os arquivos do projeto. No servidor, o EAS instala as dependências, gera o projeto nativo quando necessário, compila e assina o pacote.

O terminal fornece um link para acompanhar o processo no navegador. A compilação pode entrar em fila e levar vários minutos.

```text
Envio do projeto → Fila → Instalação de dependências
                            ↓
                  Compilação e assinatura
                            ↓
                   APK pronto para baixar
```

Se fechar o terminal, consulte os builds no painel ou execute:

```bash
npx eas-cli@latest build:list --platform android
```

> **Verificação:** aguarde o status de sucesso e confira se o artefato disponível termina em `.apk`.

---

## 9. Baixar e instalar no celular Android

### 9.1 Instalação pelo próprio celular

1. Abra no celular o link do build concluído.
2. Use a opção de instalação/download do APK na página. Se o acesso exigir autenticação, entre com uma conta autorizada.
3. Baixe o arquivo `.apk`.
4. Abra o download e toque em **Instalar**.
5. Se o Android solicitar, autorize **Instalar apps desconhecidos** para o navegador ou gerenciador de arquivos usado.
6. Conclua a instalação e abra o aplicativo pelo ícone.

Os nomes das telas variam conforme a versão do Android e o fabricante. Autorize apenas a origem usada para instalar seu APK e, após concluir, você pode desativar essa permissão.

O arquivo também pode ser transferido por cabo USB. Compartilhe o APK ou o link com os colegas que participarão dos testes, respeitando o acesso configurado no projeto EAS.

> O QR Code/link do **build** leva ao instalador. O QR Code de `npx expo start` serve para o desenvolvimento e não substitui o APK. APKs são para Android; não podem ser instalados em iPhones.

### 9.2 Alternativa: instalar por USB com ADB

Para quem já tem o Android SDK Platform-Tools instalado:

1. ative as opções do desenvolvedor e a depuração USB no aparelho;
2. conecte o cabo e autorize o computador no celular;
3. verifique a conexão e instale o arquivo baixado:

```bash
adb devices
adb install -r caminho/do/meu-app.apk
```

Substitua o caminho pelo local real do APK. O `-r` solicita reinstalar/atualizar preservando os dados quando a atualização é compatível. Para orientações de conexão, consulte a [documentação do ADB](https://developer.android.com/tools/adb).

---

## 10. Testar sem depender do computador

Agora faça uma verificação diferente do teste no Expo Go:

1. Pare o Metro no computador com `Ctrl+C`.
2. Abra o aplicativo pelo ícone próprio no celular.
3. Confira o nome, o ícone e a abertura da tela de login.
4. Entre com um usuário e teste o menu lateral.
5. Cadastre, consulte, altere e exclua um produto.
6. Feche e reabra o app para conferir a sessão e os dados.
7. Saia da conta e entre com outro usuário para testar o isolamento dos produtos.
8. Se o app usa SQLite, confira a persistência local após fechar e reabrir.
9. Teste o comportamento sem internet e depois reconecte.

**Resultado esperado:** o app abre com o Metro desligado. O Firebase precisa de rede para operações remotas; estar independente do computador não significa que todos os recursos funcionem offline.

> Os dados locais do Expo Go não são automaticamente transferidos para o aplicativo instalado. O APK tem seu próprio armazenamento. Dados remotos do Firebase podem ser recuperados ao acessar a mesma conta e o mesmo projeto.

---

## 11. Gerar e instalar uma nova versão

Alterar o código no VS Code não altera o APK já instalado. Para distribuir as mudanças neste fluxo, faça outro build.

1. Altere uma informação visível, como o texto da tela Início.
2. No `app.json`, avance `version` para `1.0.1` e `android.versionCode` para `2`.
3. Mantenha `android.package` e a chave de assinatura.
4. Execute novamente:

```bash
npx eas-cli@latest build --platform android --profile preview
```

5. Baixe e instale o novo APK sobre o anterior.
6. Confira a alteração e a preservação dos dados locais.

O exemplo usa o controle de versão no arquivo do projeto. Se o seu EAS já usa versionamento remoto ou incremento automático, siga a estratégia configurada em vez de misturar os dois métodos. Veja a [documentação de versionamento do EAS](https://docs.expo.dev/build-reference/app-versions/).

Se mudar os valores públicos do Firebase, gere outro APK para incorporar a configuração. O **EAS Update** é outra possibilidade para certas atualizações de JavaScript e recursos em apps previamente configurados; mudanças nativas continuam exigindo um novo build. Sua configuração fica fora desta aula.

---

## 12. E se eu quiser compilar no meu computador?

### 12.1 EAS Build local

É possível executar o fluxo do EAS no próprio computador. Essa alternativa **continua exigindo autenticação Expo** e pode acessar o serviço para verificar o projeto e obter credenciais.

Antes, prepare Node.js, Java/JDK compatível, Android SDK/NDK e as variáveis do ambiente Android. O EAS local oferece suporte a **macOS e Linux**. No Windows, WSL pode ser usado, mas não tem suporte oficial para esse fluxo; para a aula no Windows, o build em nuvem evita essa configuração.

Com o ambiente pronto, login feito, perfil `preview` configurado e variáveis necessárias disponíveis no ambiente local, execute:

```bash
npx eas-cli@latest build --platform android --profile preview --local --output ./meu-app.apk
```

O resultado será copiado para `meu-app.apk` na pasta atual. Variáveis EAS com visibilidade **Secret** não são disponibilizadas automaticamente no build local; confira a configuração local antes de executar. Os requisitos e limitações estão na [documentação do EAS Build local](https://docs.expo.dev/build-reference/local-builds/).

### 12.2 Build nativo sem usar o EAS

Outra possibilidade é gerar/manter o projeto Android e compilar usando Android Studio ou Gradle. Em projetos Expo que usam geração nativa, `npx expo prebuild --platform android` cria a pasta `android/`.

Esse caminho exige configurar o ambiente Android, a assinatura de release e o empacotamento do JavaScript. Apenas executar um build de debug não equivale ao APK independente deste tutorial. Para seguir esse caminho, use o [guia de build de produção local do Expo](https://docs.expo.dev/guides/local-app-production/) e as [instruções de assinatura do React Native](https://reactnative.dev/docs/signed-apk-android).

> Faça um commit ou backup antes de gerar/alterar o projeto nativo. Para esta aula, não é necessário executar `prebuild` manualmente: o EAS na nuvem faz essa etapa quando o projeto precisa dela.

---

## 13. E para publicar na Google Play?

O perfil `production` do exemplo gera um AAB:

```bash
npx eas-cli@latest build --platform android --profile production
```

Se usa Firebase, cadastre também as variáveis no ambiente **production** antes de solicitar esse build.

Gerar o AAB **não publica o aplicativo**. A publicação exige conta de desenvolvedor Google Play, cadastro do app, materiais da loja, declarações e atendimento aos requisitos vigentes. Consulte o [guia de envio para Google Play](https://docs.expo.dev/submit/android/).

Para apresentar o projeto em sala e instalar diretamente nos aparelhos, continue usando o **APK do perfil preview**.

---

## 14. Problemas comuns

| Problema | O que verificar |
|---|---|
| `package.json` não encontrado | Execute o comando dentro da pasta do app, não na pasta dos tutoriais. |
| Conta/projeto incorreto no EAS | Confira `whoami` e o vínculo do projeto antes de compilar. |
| Recebi `.aab` em vez de `.apk` | Confira `android.buildType` e use `--profile preview`. |
| APK pede conexão com Metro | Confira se o perfil não está gerando development client e instale o APK do build correto. |
| Funciona no Expo Go, mas Firebase falha no APK | Confira as seis variáveis no ambiente `preview` e gere um novo build. |
| `permission-denied` no Firestore | Confira autenticação, caminhos e regras publicadas; não abra o banco publicamente para contornar o erro. |
| Build falha na instalação de dependências | Leia o primeiro erro relevante nos logs; confira o lockfile e o resultado do `expo-doctor`. |
| Arquivo de imagem não encontrado | Confira o caminho, letras maiúsculas/minúsculas e se o arquivo foi incluído no envio. |
| Android bloqueia a instalação | Confira a permissão da origem que abriu o APK e se o aparelho tem restrições de administração. |
| “App não instalado” ao atualizar | Confira assinatura, pacote e `versionCode`. Desinstalar pode resolver conflito, mas apaga dados locais; preserve-os antes. |
| APK abre e fecha imediatamente | Consulte os logs do aparelho, por exemplo com `adb logcat`, e verifique configurações usadas no build. |
| Serviço local não responde no celular | `localhost` no aparelho aponta para o próprio celular. Use endereço acessível ao dispositivo e configuração adequada de rede/HTTPS. |
| Build demora para começar | Confira o status da fila e os limites do plano no painel Expo. |
| Alterei o código e o app instalado continua igual | Gere e instale outro APK. Fast Refresh não atualiza esse instalador. |

Para investigar falhas, consulte o [guia de erros de build e execução do Expo](https://docs.expo.dev/build-reference/troubleshooting/).

---

## 15. Atividade prática e entrega

Entregue ao professor:

1. **O APK ou um link acessível para baixá-lo**, identificando a versão.
2. Uma captura do aplicativo instalado, aberto pelo próprio ícone.
3. Uma captura ou identificação do build concluído no EAS.
4. Um relato curto dos testes de login, navegação e persistência/CRUD.
5. A resposta: **por que usamos APK nesta atividade e em que situação usaríamos AAB?**

Antes de entregar, confira:

- [ ] O arquivo entregue termina em `.apk`.
- [ ] O app abre sem Expo Go e com o Metro desligado.
- [ ] Os recursos principais foram testados no app instalado.
- [ ] O build usa o pacote Android e o projeto Firebase corretos.
- [ ] Configurações como `eas.json` e ajustes do app foram salvas no repositório do aplicativo.
- [ ] `.env.local`, keystores e senhas não foram incluídos no Git.

### Desafios

1. Personalize o nome e o ícone do app e gere um novo APK.
2. Exiba a versão do aplicativo em uma tela Sobre.
3. Distribua uma atualização e verifique se os dados locais foram preservados.
4. Compare o comportamento sem internet de uma tela com SQLite e de uma tela que consulta Firebase.
5. Explique por que a conta Expo usada para compilar é diferente das contas cadastradas pelos usuários dentro do aplicativo.

---

## Referências

- [Expo — criar o primeiro build](https://docs.expo.dev/build/setup/)
- [Expo — gerar e instalar APKs](https://docs.expo.dev/build-reference/apk/)
- [EAS — configuração do eas.json](https://docs.expo.dev/eas/json/)
- [EAS — criar e gerenciar variáveis](https://docs.expo.dev/eas/environment-variables/manage/)
- [EAS — usar variáveis nos builds](https://docs.expo.dev/eas/environment-variables/usage/)
- [Expo — variáveis públicas no aplicativo](https://docs.expo.dev/guides/environment-variables/)
- [EAS — build local](https://docs.expo.dev/build-reference/local-builds/)
- [Expo — build de produção local](https://docs.expo.dev/guides/local-app-production/)
- [React Native — assinatura Android](https://reactnative.dev/docs/signed-apk-android)
