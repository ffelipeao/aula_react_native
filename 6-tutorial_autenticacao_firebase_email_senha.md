# Tutorial 6 — App com login no Firebase e menu lateral

Nesta aula vamos criar **um aplicativo novo**, chamado **MeuAppFirebase**, com:

- tela de **login** por e-mail e senha usando o **Firebase Authentication**;
- tela **Início** exibida após o login;
- **menu lateral** aberto pelo ícone de três barrinhas (☰) com as opções **Sobre o usuário** e **Contato**;
- estilos em um **arquivo externo** (`src/styles/styles.js`);
- credenciais do Firebase em um arquivo **`.env.local`**, que **não vai para o GitHub**.

```text
Abrir app → Firebase verifica a sessão
              ├── sem usuário → Login
              └── com usuário → Início ☰ ── Sobre o usuário
                                         └── Contato
```

---

## 1. Criar o projeto

No terminal, na pasta onde você guarda seus projetos:

```bash
npx create-expo-app@latest MeuAppFirebase --template blank
cd MeuAppFirebase
```

Instale as bibliotecas de navegação (menu lateral) e do Firebase:

```bash
npm install @react-navigation/native @react-navigation/drawer
npx expo install react-native-screens react-native-safe-area-context react-native-gesture-handler react-native-reanimated react-native-worklets
npx expo install firebase @react-native-async-storage/async-storage
```

> `npx expo install` escolhe versões compatíveis com o SDK do Expo do projeto. O `babel-preset-expo` já configura o Reanimated; não é preciso alterar o `babel.config.js`.

Crie as pastas e arquivos abaixo (vazios por enquanto):

```text
MeuAppFirebase/
├── App.js                  ← já existe, será substituído
├── .env.local              ← credenciais (NÃO versionar)
├── .env.example            ← modelo sem valores (pode versionar)
└── src/
    ├── services/firebase.js
    ├── styles/styles.js
    └── screens/
        ├── LoginScreen.js
        ├── HomeScreen.js
        ├── SobreScreen.js
        └── ContatoScreen.js
```

---

## 2. Configurar o Firebase (passo a passo)

### 2.1 Criar o projeto no console

1. Acesse <https://console.firebase.google.com/> com sua conta Google.
2. Clique em **Criar projeto** → nome `MeuAppFirebase` → o Google Analytics é opcional → **Criar**.

### 2.2 Ativar login por e-mail e senha

1. No menu à esquerda: **Criação (Build) → Authentication → Começar**.
2. Aba **Método de login (Sign-in method)** → **E-mail/senha** → ative a primeira opção → **Salvar**.

### 2.3 Cadastrar um usuário de teste

1. Ainda em **Authentication**, aba **Usuários (Users)** → **Adicionar usuário**.
2. Informe um e-mail (ex.: `aluno@teste.com`) e uma senha com pelo menos 6 caracteres.

É com esse usuário que você vai entrar no app.

### 2.4 Copiar as credenciais

1. Clique na **engrenagem ⚙️ → Configurações do projeto**.
2. Na aba **Geral**, desça até **Seus apps** e clique no ícone **Web `</>`**.
3. Dê um apelido (ex.: `MeuAppFirebase`), **não** marque Hosting e clique em **Registrar app**.
4. O console mostra um trecho parecido com este — são esses valores que você vai copiar:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "meuappfirebase-xxxx.firebaseapp.com",
  projectId: "meuappfirebase-xxxx",
  storageBucket: "meuappfirebase-xxxx.firebasestorage.app",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abc123"
};
```

> Registramos um app **Web** porque o Expo Go usa o **Firebase JavaScript SDK**, que funciona em Android e iOS. Para rever essas credenciais depois: ⚙️ → **Configurações do projeto** → **Seus apps**.

### 2.5 Colar as credenciais no projeto

Na **raiz do projeto** (mesma pasta do `package.json`), crie o arquivo **`.env.local`** e cole cada valor do `firebaseConfig` na linha correspondente, **sem aspas**:

```env
EXPO_PUBLIC_FIREBASE_API_KEY=AIzaSy...
EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN=meuappfirebase-xxxx.firebaseapp.com
EXPO_PUBLIC_FIREBASE_PROJECT_ID=meuappfirebase-xxxx
EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET=meuappfirebase-xxxx.firebasestorage.app
EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=1234567890
EXPO_PUBLIC_FIREBASE_APP_ID=1:1234567890:web:abc123
```

| No `firebaseConfig` | No `.env.local` |
|---|---|
| `apiKey` | `EXPO_PUBLIC_FIREBASE_API_KEY` |
| `authDomain` | `EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN` |
| `projectId` | `EXPO_PUBLIC_FIREBASE_PROJECT_ID` |
| `storageBucket` | `EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET` |
| `messagingSenderId` | `EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID` |
| `appId` | `EXPO_PUBLIC_FIREBASE_APP_ID` |

O prefixo `EXPO_PUBLIC_` faz o Expo entregar a variável ao código do app.

### 2.6 Não versionar as chaves

1. Abra o `.gitignore` do projeto e confirme que existe a linha abaixo (o template do Expo já traz). Se não existir, acrescente:

   ```gitignore
   .env*.local
   ```

2. Crie o **`.env.example`**, que vai para o GitHub **sem os valores**, só para mostrar quais variáveis são necessárias:

   ```env
   EXPO_PUBLIC_FIREBASE_API_KEY=
   EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN=
   EXPO_PUBLIC_FIREBASE_PROJECT_ID=
   EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET=
   EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
   EXPO_PUBLIC_FIREBASE_APP_ID=
   ```

3. Confira se o Git está ignorando o arquivo:

   ```bash
   git check-ignore -v .env.local
   ```

   Se aparecer uma linha citando o `.gitignore`, está correto. O `.env.local` também **não** deve aparecer em `git status`.

> **Já fez commit do `.env.local` sem querer?** Execute `git rm --cached .env.local`, faça um novo commit e, por precaução, gere novas credenciais no console.
>
> A configuração Web do Firebase identifica o projeto, mas não é uma senha de administrador: ela acaba incluída no app instalado. Mesmo assim, mantê-la fora do repositório é uma boa prática. A proteção real dos dados é feita pelo Authentication e pelas regras de segurança (Security Rules).

---

## 3. Conectar o app ao Firebase

`src/services/firebase.js`:

```js
import AsyncStorage from '@react-native-async-storage/async-storage';
import { getApp, getApps, initializeApp } from 'firebase/app';
import { getAuth, getReactNativePersistence, initializeAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: process.env.EXPO_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.EXPO_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.EXPO_PUBLIC_FIREBASE_APP_ID,
};

const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApp();

let auth;
try {
  // Guarda a sessão no aparelho: o usuário continua logado ao reabrir o app.
  auth = initializeAuth(app, { persistence: getReactNativePersistence(AsyncStorage) });
} catch {
  // Durante o recarregamento automático do Expo o Auth já pode existir.
  auth = getAuth(app);
}

export { auth };
```

Os valores vêm do `.env.local`, portanto **nenhuma chave aparece no código**.

---

## 4. Arquivo externo de estilos

No React Native os estilos são objetos JavaScript criados com `StyleSheet`. Deixando todos em um único arquivo, as telas ficam mais limpas e as cores podem ser alteradas em um só lugar.

`src/styles/styles.js`:

```js
import { StyleSheet } from 'react-native';

const cores = {
  primaria: '#2563EB',
  fundo: '#F1F5F9',
  texto: '#1E293B',
  erro: '#B91C1C',
};

export default StyleSheet.create({
  container: { flex: 1, backgroundColor: cores.fundo, padding: 20 },
  centro: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  titulo: { fontSize: 24, fontWeight: 'bold', color: cores.primaria, marginBottom: 16 },
  texto: { fontSize: 16, color: cores.texto, marginBottom: 8 },
  campo: {
    backgroundColor: '#FFF', borderWidth: 1, borderColor: '#CBD5E1',
    borderRadius: 8, padding: 12, marginBottom: 12, fontSize: 16,
  },
  botao: { backgroundColor: cores.primaria, borderRadius: 8, padding: 14, alignItems: 'center' },
  textoBotao: { color: '#FFF', fontWeight: 'bold', fontSize: 16 },
  erro: { color: cores.erro, marginBottom: 12 },
  cartao: { backgroundColor: '#FFF', borderRadius: 8, padding: 16, marginBottom: 12 },
  botaoSair: { marginRight: 16 },
  textoSair: { color: cores.primaria, fontWeight: 'bold' },
});
```

Uso nas telas: `import styles from '../styles/styles';` e depois `style={styles.titulo}`.

---

## 5. Tela de login

`src/screens/LoginScreen.js`:

```jsx
import { useState } from 'react';
import { ActivityIndicator, Pressable, Text, TextInput } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { signInWithEmailAndPassword } from 'firebase/auth';
import { auth } from '../services/firebase';
import styles from '../styles/styles';

export default function LoginScreen() {
  const [email, setEmail] = useState('');
  const [senha, setSenha] = useState('');
  const [erro, setErro] = useState('');
  const [carregando, setCarregando] = useState(false);

  async function entrar() {
    if (!email.trim() || !senha) {
      setErro('Informe e-mail e senha.');
      return;
    }
    setErro('');
    setCarregando(true);
    try {
      await signInWithEmailAndPassword(auth, email.trim(), senha);
      // Não é preciso navegar: o App.js percebe o login e mostra a Home.
    } catch (e) {
      setErro(e.code === 'auth/network-request-failed'
        ? 'Sem conexão com a internet.'
        : 'E-mail ou senha inválidos.');
    } finally {
      setCarregando(false);
    }
  }

  return (
    <SafeAreaView style={[styles.container, { justifyContent: 'center' }]}>
      <Text style={styles.titulo}>Entrar</Text>

      <TextInput
        style={styles.campo}
        placeholder="E-mail"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
      />
      <TextInput
        style={styles.campo}
        placeholder="Senha"
        value={senha}
        onChangeText={setSenha}
        secureTextEntry
      />

      {erro !== '' && <Text style={styles.erro}>{erro}</Text>}

      <Pressable style={styles.botao} onPress={entrar} disabled={carregando}>
        {carregando
          ? <ActivityIndicator color="#FFF" />
          : <Text style={styles.textoBotao}>Entrar</Text>}
      </Pressable>
    </SafeAreaView>
  );
}
```

> A mensagem "E-mail ou senha inválidos" é genérica de propósito: assim o app não revela se um e-mail está ou não cadastrado.

---

## 6. Telas internas

### 6.1 Início

`src/screens/HomeScreen.js`:

```jsx
import { Text, View } from 'react-native';
import { auth } from '../services/firebase';
import styles from '../styles/styles';

export default function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Bem-vindo!</Text>
      <Text style={styles.texto}>Você entrou como {auth.currentUser?.email}.</Text>
      <Text style={styles.texto}>Toque no ☰ no canto superior esquerdo para abrir o menu.</Text>
    </View>
  );
}
```

### 6.2 Sobre o usuário

`src/screens/SobreScreen.js`:

```jsx
import { Text, View } from 'react-native';
import { auth } from '../services/firebase';
import styles from '../styles/styles';

export default function SobreScreen() {
  const usuario = auth.currentUser;

  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Sobre o usuário</Text>
      <View style={styles.cartao}>
        <Text style={styles.texto}>E-mail: {usuario?.email}</Text>
        <Text style={styles.texto}>ID (uid): {usuario?.uid}</Text>
        <Text style={styles.texto}>Conta criada em: {usuario?.metadata.creationTime}</Text>
        <Text style={styles.texto}>Último acesso: {usuario?.metadata.lastSignInTime}</Text>
      </View>
    </View>
  );
}
```

### 6.3 Contato

`src/screens/ContatoScreen.js`:

```jsx
import { Linking, Pressable, Text, View } from 'react-native';
import styles from '../styles/styles';

export default function ContatoScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Contato</Text>
      <View style={styles.cartao}>
        <Text style={styles.texto}>E-mail: suporte@meuapp.com</Text>
        <Text style={styles.texto}>Telefone: (21) 99999-9999</Text>
      </View>
      <Pressable style={styles.botao} onPress={() => Linking.openURL('mailto:suporte@meuapp.com')}>
        <Text style={styles.textoBotao}>Enviar e-mail</Text>
      </Pressable>
    </View>
  );
}
```

---

## 7. App.js: login ou menu

Substitua todo o conteúdo de `App.js`:

```jsx
import 'react-native-gesture-handler';
import { useEffect, useState } from 'react';
import { ActivityIndicator, Pressable, Text, View } from 'react-native';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { NavigationContainer } from '@react-navigation/native';
import { createDrawerNavigator } from '@react-navigation/drawer';
import { onAuthStateChanged, signOut } from 'firebase/auth';

import { auth } from './src/services/firebase';
import styles from './src/styles/styles';
import LoginScreen from './src/screens/LoginScreen';
import HomeScreen from './src/screens/HomeScreen';
import SobreScreen from './src/screens/SobreScreen';
import ContatoScreen from './src/screens/ContatoScreen';

const Drawer = createDrawerNavigator();

function BotaoSair() {
  return (
    <Pressable style={styles.botaoSair} onPress={() => signOut(auth)}>
      <Text style={styles.textoSair}>Sair</Text>
    </Pressable>
  );
}

export default function App() {
  const [usuario, setUsuario] = useState(null);
  const [verificando, setVerificando] = useState(true);

  useEffect(() => {
    // Executa ao abrir o app e sempre que alguém entra ou sai.
    const cancelar = onAuthStateChanged(auth, (u) => {
      setUsuario(u);
      setVerificando(false);
    });
    return cancelar;
  }, []);

  if (verificando) {
    return (
      <View style={styles.centro}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  return (
    <SafeAreaProvider>
      {usuario ? (
        <NavigationContainer>
          <Drawer.Navigator screenOptions={{ headerRight: () => <BotaoSair /> }}>
            <Drawer.Screen name="Inicio" component={HomeScreen} options={{ title: 'Início' }} />
            <Drawer.Screen name="Sobre" component={SobreScreen} options={{ title: 'Sobre o usuário' }} />
            <Drawer.Screen name="Contato" component={ContatoScreen} />
          </Drawer.Navigator>
        </NavigationContainer>
      ) : (
        <LoginScreen />
      )}
    </SafeAreaProvider>
  );
}
```

Como funciona:

- `onAuthStateChanged` avisa quando o usuário entra ou sai; o `App.js` então troca entre **Login** e **menu**.
- O `Drawer.Navigator` cria o cabeçalho com o ícone **☰**, que abre o menu lateral (também abre arrastando da borda esquerda).
- Cada `Drawer.Screen` vira um item do menu. O `title` é o texto exibido.
- O botão **Sair** no canto direito chama `signOut`, e o app volta para o login.

---

## 8. Executar e testar

```bash
npx expo start --clear
```

> Use `--clear` sempre que criar ou alterar o `.env.local`; caso contrário, o Expo pode continuar usando valores antigos.

Abra no **Expo Go** pelo QR code (o celular precisa de internet) e confira:

1. O app abre na tela de login.
2. Senha errada → aparece "E-mail ou senha inválidos".
3. Entre com o usuário criado no passo 2.3 → abre a tela **Início**.
4. Toque no **☰** → o menu mostra **Início**, **Sobre o usuário** e **Contato**.
5. **Sobre o usuário** mostra o e-mail e o `uid` (compare com **Authentication → Usuários** no console).
6. Feche e reabra o app → você continua logado.
7. Toque em **Sair** → volta para o login.
8. Altere a cor `primaria` em `styles.js` e veja todas as telas mudarem.
9. Rode `git status` e confirme que o `.env.local` **não** aparece.

---

## 9. Problemas comuns

| Problema | Solução |
|---|---|
| `auth/invalid-api-key` ou `auth/configuration-not-found` | Revise o `.env.local` (nomes das variáveis, sem aspas, na raiz do projeto) e rode `npx expo start --clear`. |
| Sempre "E-mail ou senha inválidos" | Confira se o provedor **E-mail/senha** está ativado e se o usuário existe em **Authentication → Usuários**. |
| `auth/operation-not-allowed` | Ative **E-mail/senha** em **Authentication → Método de login**. |
| Erro com `reanimated` ou `worklets` | Execute de novo `npx expo install react-native-reanimated react-native-worklets` e reinicie com `--clear`. |
| `Unable to resolve @react-navigation/...` | Execute novamente os comandos de instalação do passo 1. |
| `.env.local` aparece no `git status` | Confira o `.gitignore` (passo 2.6) e, se já foi commitado, use `git rm --cached .env.local`. |

---

## 10. Desafios

1. Adicione um botão **Criar conta** na tela de login usando `createUserWithEmailAndPassword(auth, email, senha)`. A solução completa está no [Tutorial 6.1](6.1-tutorial_cadastro_usuarios_firebase.md).
2. Adicione um link **Esqueci minha senha** com `sendPasswordResetEmail(auth, email)`.
3. Crie uma quarta opção no menu (ex.: **Configurações**).
4. Mostre as datas da tela **Sobre o usuário** no formato brasileiro com `new Date(...).toLocaleString('pt-BR')`.

## Referências

- [Firebase no Expo](https://docs.expo.dev/guides/using-firebase/)
- [Variáveis de ambiente no Expo](https://docs.expo.dev/guides/environment-variables/)
- [Firebase Authentication — e-mail e senha](https://firebase.google.com/docs/auth/web/password-auth)
- [React Navigation — Drawer](https://reactnavigation.org/docs/drawer-navigator)
- [Estilos no React Native](https://reactnative.dev/docs/style)
