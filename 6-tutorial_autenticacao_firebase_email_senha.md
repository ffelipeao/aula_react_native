# Tutorial — MeuCadastroDeProdutos com login pelo Firebase

Vamos criar **um novo aplicativo**, chamado **MeuCadastroDeProdutos**, usando React Native e Expo. Ele terá uma tela de login por e-mail e senha e uma área interna para cadastrar, consultar e alterar produtos. Não é necessário copiar o projeto das aulas anteriores.

Para simplificar, usaremos apenas duas telas, exibidas conforme a sessão do usuário:

```text
Abrir app → verificar sessão → Login (entrar ou criar conta)
                                  ↓
                            Área de produtos
                       cadastrar → listar → alterar
                                  ↓
                            Sair → Login
```

O **Firebase Authentication** cuida das contas e senhas. Os produtos ficam no **SQLite do dispositivo**, separados pelo identificador (`uid`) da conta. Não há sincronização dos produtos entre aparelhos neste exemplo.

> **Sobre o CSS externo:** React Native para Android e iOS usa estilos JavaScript, em vez de importar um `.css` convencional. Para facilitar a manutenção, todos os estilos deste tutorial ficam no arquivo externo `src/styles/styles.js`, usando `StyleSheet`. Assim, cores, tamanhos e espaçamentos podem ser alterados em um único lugar. Veja a [documentação de estilos do React Native](https://reactnative.dev/docs/style).

## 1. Criar o novo aplicativo

Com Node.js LTS instalado, execute:

```bash
npx create-expo-app@latest MeuCadastroDeProdutos --template blank
cd MeuCadastroDeProdutos
npx expo install firebase @react-native-async-storage/async-storage expo-sqlite react-native-safe-area-context
```

O template `blank` usa JavaScript e um arquivo `App.js`. Não precisamos instalar uma biblioteca de navegação: o estado de autenticação determina qual tela aparece. Usaremos o Expo Go em Android ou iOS; este tutorial não configura SQLite para execução no navegador.

Crie a seguinte estrutura dentro do projeto:

```text
MeuCadastroDeProdutos/
├── App.js
├── .env.local
├── .env.example
└── src/
    ├── contexts/AuthContext.js
    ├── database/database.js
    ├── screens/LoginScreen.js
    ├── screens/ProductsScreen.js
    ├── services/firebase.js
    ├── styles/styles.js
    └── utils/authErrors.js
```

## 2. Criar e configurar o projeto Firebase

O Firebase oferece serviços prontos para aplicativos. Nesta aula usaremos apenas Authentication, sem Firestore nem Hosting.

1. Acesse o [console do Firebase](https://console.firebase.google.com/) com sua conta Google.
2. Crie um projeto chamado `MeuCadastroDeProdutos`. O Google Analytics é opcional.
3. Na visão geral, selecione o ícone **Web** (`</>`).
4. Registre um app com o apelido `MeuCadastroDeProdutos`. Não é necessário ativar Hosting.
5. Copie o objeto `firebaseConfig` mostrado pelo console.

Registramos um app Web porque usamos o **Firebase JavaScript SDK**, compatível com Expo Go, mesmo executando em Android ou iOS. Consulte o [guia do Expo para Firebase](https://docs.expo.dev/guides/using-firebase/).

Use o plano Spark para esta atividade. Confira as [cotas e condições atuais](https://firebase.google.com/pricing) antes de usar o serviço em produção.

> A conta Google do desenvolvedor serve para acessar o console. As contas de acesso ao aplicativo serão cadastradas com e-mail e senha no Firebase Authentication.

## 3. Ativar a autenticação por e-mail e senha

1. Abra **Authentication** no console.
2. Selecione **Começar**, caso solicitado.
3. Em **Método de login / Sign-in method**, selecione **E-mail/senha**.
4. Ative o login por e-mail e senha e salve. O login por link de e-mail não será usado.

Sem essa configuração, o cadastro pode retornar `auth/operation-not-allowed`.

## 4. Configurar as variáveis do projeto

Crie o arquivo `.env.local` na raiz do projeto:

```env
EXPO_PUBLIC_FIREBASE_API_KEY=cole-a-api-key
EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN=seu-projeto.firebaseapp.com
EXPO_PUBLIC_FIREBASE_PROJECT_ID=seu-projeto
EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET=seu-projeto.firebasestorage.app
EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=cole-o-sender-id
EXPO_PUBLIC_FIREBASE_APP_ID=cole-o-app-id
```

Substitua cada exemplo pelo valor correspondente do seu `firebaseConfig`. Não use aspas nem espaços ao redor do sinal de igual.

O prefixo `EXPO_PUBLIC_` permite que o Expo disponibilize a variável no código executado pelo aplicativo. Isso também significa que os valores ficam visíveis no pacote final.

> A configuração de um aplicativo Firebase identifica o projeto, mas não funciona como uma senha administrativa. Mesmo assim, nunca coloque chaves privadas, senhas ou credenciais de servidor em variáveis `EXPO_PUBLIC_`. A proteção dos dados Firebase deve ser feita com Authentication, Security Rules e App Check, quando aplicável.

Acrescente `.env.local` ao `.gitignore`:

```gitignore
.env.local
```

Crie também `.env.example`, que pode ser enviado ao GitHub sem os valores da sua turma:

```env
EXPO_PUBLIC_FIREBASE_API_KEY=
EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN=
EXPO_PUBLIC_FIREBASE_PROJECT_ID=
EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET=
EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
EXPO_PUBLIC_FIREBASE_APP_ID=
```

Depois de criar ou alterar as variáveis, reinicie o Expo:

```bash
npx expo start --clear
```

## 5. Inicializar o Firebase

Crie a pasta `src/services` e o arquivo `src/services/firebase.js`:

```js
import AsyncStorage from '@react-native-async-storage/async-storage';
import { getApp, getApps, initializeApp } from 'firebase/app';
import {
  getAuth,
  getReactNativePersistence,
  initializeAuth,
} from 'firebase/auth';

const firebaseConfig = {
  apiKey: process.env.EXPO_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.EXPO_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.EXPO_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.EXPO_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.EXPO_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.EXPO_PUBLIC_FIREBASE_APP_ID,
};

const app = getApps().length === 0
  ? initializeApp(firebaseConfig)
  : getApp();

let auth;

try {
  auth = initializeAuth(app, {
    persistence: getReactNativePersistence(AsyncStorage),
  });
} catch (error) {
  if (error.code === 'auth/already-initialized') {
    auth = getAuth(app);
  } else {
    throw error;
  }
}

export { auth };
```

O teste com `getApps()` evita inicializar o aplicativo Firebase mais de uma vez durante as atualizações automáticas do Expo. O bloco `try/catch` faz o mesmo para o serviço de autenticação.

`getReactNativePersistence(AsyncStorage)` informa onde o Firebase deve guardar a sessão. A senha não é armazenada por esse código.

## 6. Criar o contexto de autenticação

O contexto permitirá que diferentes telas descubram qual usuário está conectado e usem as ações de entrar, cadastrar e sair.

Crie a pasta `src/contexts` e o arquivo `src/contexts/AuthContext.js`:

```jsx
import { createContext, useContext, useEffect, useState } from 'react';
import {
  createUserWithEmailAndPassword,
  onAuthStateChanged,
  signInWithEmailAndPassword,
  signOut,
} from 'firebase/auth';

import { auth } from '../services/firebase';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [usuario, setUsuario] = useState(null);
  const [carregando, setCarregando] = useState(true);

  useEffect(() => {
    const cancelarObservacao = onAuthStateChanged(auth, (usuarioFirebase) => {
      setUsuario(usuarioFirebase);
      setCarregando(false);
    });

    return cancelarObservacao;
  }, []);

  async function entrar(email, senha) {
    return signInWithEmailAndPassword(auth, email.trim(), senha);
  }

  async function criarConta(email, senha) {
    return createUserWithEmailAndPassword(auth, email.trim(), senha);
  }

  async function sair() {
    return signOut(auth);
  }

  return (
    <AuthContext.Provider
      value={{ usuario, carregando, entrar, criarConta, sair }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const contexto = useContext(AuthContext);

  if (!contexto) {
    throw new Error('useAuth deve ser usado dentro de AuthProvider.');
  }

  return contexto;
}
```

O observador `onAuthStateChanged` é executado quando a verificação inicial termina e sempre que alguém entra ou sai. Por isso, não precisamos navegar manualmente para a tela inicial depois do login.

## 7. Traduzir os erros mais comuns

Crie `src/utils/authErrors.js`:

```js
export function obterMensagemDeAutenticacao(codigo) {
  const mensagens = {
    'auth/email-already-in-use': 'Este e-mail já possui uma conta.',
    'auth/invalid-email': 'Informe um e-mail válido.',
    'auth/invalid-credential': 'E-mail ou senha incorretos.',
    'auth/user-disabled': 'Esta conta foi desativada.',
    'auth/weak-password': 'A senha não atende aos requisitos de segurança.',
    'auth/too-many-requests': 'Muitas tentativas. Aguarde e tente novamente.',
    'auth/network-request-failed': 'Não foi possível acessar a internet.',
    'auth/operation-not-allowed': 'Ative o login por e-mail e senha no Firebase.',
  };

  return mensagens[codigo] ?? 'Não foi possível concluir a autenticação.';
}
```

Mensagens genéricas no login evitam revelar se determinado e-mail possui uma conta. Essa prática reduz a possibilidade de enumeração de usuários.

## 8. Separar os estilos das telas

Crie `src/styles/styles.js`. Este é o arquivo externo de estilos compartilhado por todas as telas:

```js
import { StyleSheet } from 'react-native';

export default StyleSheet.create({
  container: { flex: 1, backgroundColor: '#F1F5F9' },
  conteudo: { padding: 20, paddingBottom: 40 },
  centro: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  titulo: { fontSize: 25, fontWeight: 'bold', color: '#1E3A8A', marginBottom: 12 },
  texto: { color: '#475569', marginBottom: 12 },
  rotulo: { color: '#334155', fontWeight: 'bold', marginBottom: 6 },
  campo: {
    backgroundColor: '#FFFFFF', borderWidth: 1, borderColor: '#CBD5E1',
    borderRadius: 8, padding: 12, marginBottom: 14, fontSize: 16,
  },
  botao: {
    backgroundColor: '#2563EB', borderRadius: 8, padding: 14,
    alignItems: 'center', marginBottom: 12,
  },
  secundario: { backgroundColor: '#475569' },
  desativado: { opacity: 0.6 },
  textoBotao: { color: '#FFFFFF', fontWeight: 'bold', fontSize: 16 },
  mensagem: { color: '#B91C1C', marginBottom: 12 },
  cartao: { backgroundColor: '#FFFFFF', padding: 16, borderRadius: 8, marginBottom: 12 },
  nomeProduto: { fontSize: 18, fontWeight: 'bold', marginBottom: 8 },
});
```

Nas telas, `import styles from '../styles/styles'` carrega esses estilos. Por exemplo, `style={styles.campo}` aplica o estilo de um campo. Para combinar estilos, use um array: `style={[styles.botao, styles.secundario]}`.

## 9. Criar a tela de login

Crie `src/screens/LoginScreen.js`:

```jsx
import { useState } from 'react';
import { ActivityIndicator, Pressable, ScrollView, Text, TextInput } from 'react-native';
import { useAuth } from '../contexts/AuthContext';
import { obterMensagemDeAutenticacao } from '../utils/authErrors';
import styles from '../styles/styles';

export default function LoginScreen() {
  const { entrar, criarConta } = useAuth();
  const [email, setEmail] = useState('');
  const [senha, setSenha] = useState('');
  const [enviando, setEnviando] = useState(false);
  const [mensagem, setMensagem] = useState('');

  async function autenticar(cadastrar = false) {
    if (enviando) return;
    setMensagem('');
    if (!email.trim() || !senha) {
      setMensagem('Informe o e-mail e a senha.');
      return;
    }
    try {
      setEnviando(true);
      if (cadastrar) {
        await criarConta(email, senha);
      } else {
        await entrar(email, senha);
      }
    } catch (erro) {
      setMensagem(obterMensagemDeAutenticacao(erro.code));
    } finally {
      setEnviando(false);
    }
  }

  return (
    <ScrollView style={styles.container} contentContainerStyle={styles.conteudo}
      keyboardShouldPersistTaps="handled">
      <Text style={styles.titulo}>MeuCadastroDeProdutos</Text>
      <Text style={styles.texto}>Entre para acessar seus produtos.</Text>
      <Text style={styles.rotulo}>E-mail</Text>
      <TextInput style={styles.campo} value={email} onChangeText={setEmail}
        placeholder="aluno@exemplo.com" keyboardType="email-address"
        autoCapitalize="none" autoCorrect={false} editable={!enviando} />
      <Text style={styles.rotulo}>Senha</Text>
      <TextInput style={styles.campo} value={senha} onChangeText={setSenha}
        placeholder="Digite sua senha" secureTextEntry autoCapitalize="none"
        autoCorrect={false} editable={!enviando} />
      <Text style={styles.texto}>
        Primeiro acesso? Informe seu e-mail e uma senha com pelo menos 6 caracteres
        e toque em Criar conta. Se houver uma política mais exigente no Firebase,
        a senha também deverá atendê-la.
      </Text>
      {mensagem !== '' && <Text style={styles.mensagem}>{mensagem}</Text>}
      {enviando && <ActivityIndicator />}
      <Pressable style={[styles.botao, enviando && styles.desativado]}
        disabled={enviando} onPress={() => autenticar()}>
        <Text style={styles.textoBotao}>Entrar</Text>
      </Pressable>
      <Pressable style={[styles.botao, styles.secundario, enviando && styles.desativado]}
        disabled={enviando} onPress={() => autenticar(true)}>
        <Text style={styles.textoBotao}>Criar conta</Text>
      </Pressable>
    </ScrollView>
  );
}
```

Para deixar o exemplo simples, a mesma tela permite entrar e criar uma conta. O Firebase faz login automaticamente após um cadastro bem-sucedido. Não precisamos chamar uma função de navegação: `onAuthStateChanged` atualiza o usuário e o `App.js` troca a tela.

## 10. Criar o banco local de produtos

Crie `src/database/database.js`:

```js
export async function initializeDatabase(db) {
  await db.execAsync(`
    PRAGMA journal_mode = WAL;
    CREATE TABLE IF NOT EXISTS produtos (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      usuario_uid TEXT NOT NULL,
      nome TEXT NOT NULL,
      preco REAL NOT NULL,
      quantidade INTEGER NOT NULL
    );
  `);
}
```

O banco é novo e não exige migração do projeto anterior. `usuario_uid` identifica a conta que cadastrou o produto. Todas as consultas e alterações abaixo filtram por esse campo.

## 11. Criar a área interna para cadastrar e alterar produtos

Crie `src/screens/ProductsScreen.js`:

```jsx
import { useEffect, useState } from 'react';
import { Pressable, ScrollView, Text, TextInput, View } from 'react-native';
import { useSQLiteContext } from 'expo-sqlite';
import { useAuth } from '../contexts/AuthContext';
import styles from '../styles/styles';

export default function ProductsScreen() {
  const db = useSQLiteContext();
  const { usuario, sair } = useAuth();
  const [produtos, setProdutos] = useState([]);
  const [idEdicao, setIdEdicao] = useState(null);
  const [nome, setNome] = useState('');
  const [preco, setPreco] = useState('');
  const [quantidade, setQuantidade] = useState('');
  const [ocupado, setOcupado] = useState(false);
  const [mensagem, setMensagem] = useState('');

  async function carregarProdutos() {
    const resultado = await db.getAllAsync(
      'SELECT id, nome, preco, quantidade FROM produtos WHERE usuario_uid = ? ORDER BY nome',
      usuario.uid
    );
    setProdutos(resultado);
  }

  useEffect(() => {
    carregarProdutos().catch(() => setMensagem('Não foi possível carregar os produtos.'));
  }, [db, usuario.uid]);

  function limparFormulario() {
    setIdEdicao(null);
    setNome('');
    setPreco('');
    setQuantidade('');
  }

  function editar(produto) {
    setMensagem('');
    setIdEdicao(produto.id);
    setNome(produto.nome);
    setPreco(String(produto.preco));
    setQuantidade(String(produto.quantidade));
  }

  async function salvar() {
    if (ocupado) return;
    const valor = Number(preco.trim().replace(',', '.'));
    const unidades = Number(quantidade.trim());
    if (!nome.trim() || !preco.trim() || !quantidade.trim()
      || !Number.isFinite(valor) || valor < 0
      || !Number.isSafeInteger(unidades) || unidades < 0) {
      setMensagem('Informe nome, preço não negativo e quantidade inteira não negativa.');
      return;
    }

    try {
      setOcupado(true);
      setMensagem('');
      if (idEdicao !== null) {
        await db.runAsync(
          'UPDATE produtos SET nome = ?, preco = ?, quantidade = ? WHERE id = ? AND usuario_uid = ?',
          nome.trim(), valor, unidades, idEdicao, usuario.uid
        );
      } else {
        await db.runAsync(
          'INSERT INTO produtos (usuario_uid, nome, preco, quantidade) VALUES (?, ?, ?, ?)',
          usuario.uid, nome.trim(), valor, unidades
        );
      }
      limparFormulario();
      await carregarProdutos();
      setMensagem('Produto salvo.');
    } catch (erro) {
      setMensagem('Não foi possível concluir a operação. Confira a lista antes de tentar novamente.');
    } finally {
      setOcupado(false);
    }
  }

  async function encerrarSessao() {
    if (ocupado) return;
    try {
      setOcupado(true);
      await sair();
    } catch (erro) {
      setMensagem('Não foi possível sair. Tente novamente.');
    } finally {
      setOcupado(false);
    }
  }

  return (
    <ScrollView style={styles.container} contentContainerStyle={styles.conteudo}
      keyboardShouldPersistTaps="handled">
      <Text style={styles.titulo}>Meus produtos</Text>
      <Text style={styles.texto}>Conectado: {usuario.email}</Text>
      <Pressable style={[styles.botao, styles.secundario]} disabled={ocupado}
        onPress={encerrarSessao}>
        <Text style={styles.textoBotao}>Sair</Text>
      </Pressable>

      <Text style={styles.titulo}>
        {idEdicao === null ? 'Cadastrar produto' : 'Alterar produto'}
      </Text>
      <Text style={styles.rotulo}>Nome</Text>
      <TextInput style={styles.campo} value={nome} onChangeText={setNome}
        editable={!ocupado} placeholder="Ex.: Caderno" />
      <Text style={styles.rotulo}>Preço (R$)</Text>
      <TextInput style={styles.campo} value={preco} onChangeText={setPreco}
        editable={!ocupado} keyboardType="decimal-pad" placeholder="Ex.: 12,50" />
      <Text style={styles.rotulo}>Quantidade</Text>
      <TextInput style={styles.campo} value={quantidade} onChangeText={setQuantidade}
        editable={!ocupado} keyboardType="number-pad" placeholder="Ex.: 10" />
      {mensagem !== '' && <Text style={styles.mensagem}>{mensagem}</Text>}
      <Pressable style={[styles.botao, ocupado && styles.desativado]}
        disabled={ocupado} onPress={salvar}>
        <Text style={styles.textoBotao}>
          {ocupado ? 'Aguarde...' : idEdicao === null ? 'Cadastrar' : 'Salvar alterações'}
        </Text>
      </Pressable>
      {idEdicao !== null && (
        <Pressable style={[styles.botao, styles.secundario]} disabled={ocupado}
          onPress={limparFormulario}>
          <Text style={styles.textoBotao}>Cancelar edição</Text>
        </Pressable>
      )}

      <Text style={styles.titulo}>Produtos cadastrados</Text>
      {produtos.length === 0 && <Text style={styles.texto}>Nenhum produto cadastrado.</Text>}
      {produtos.map((produto) => (
        <View key={produto.id} style={styles.cartao}>
          <Text style={styles.nomeProduto}>{produto.nome}</Text>
          <Text style={styles.texto}>
            R$ {produto.preco.toFixed(2).replace('.', ',')} | Quantidade: {produto.quantidade}
          </Text>
          <Pressable style={styles.botao} disabled={ocupado} onPress={() => editar(produto)}>
            <Text style={styles.textoBotao}>Alterar</Text>
          </Pressable>
        </View>
      ))}
    </ScrollView>
  );
}
```

Ao tocar em **Alterar**, o formulário acima da lista recebe os dados do produto. Role até ele, modifique os campos e toque em **Salvar alterações**. O `UPDATE` mantém o mesmo registro; o `INSERT` é usado somente para novos produtos. **Cancelar edição** limpa o formulário e volta ao cadastro.

Os marcadores `?` passam os valores separadamente do SQL. O filtro por `usuario_uid` também está no `UPDATE`, para que a alteração corresponda à conta conectada.

## 12. Montar o aplicativo e controlar o acesso

Substitua o conteúdo de `App.js`:

```jsx
import { ActivityIndicator, Text, View } from 'react-native';
import { SafeAreaProvider, SafeAreaView } from 'react-native-safe-area-context';
import { SQLiteProvider } from 'expo-sqlite';
import { AuthProvider, useAuth } from './src/contexts/AuthContext';
import { initializeDatabase } from './src/database/database';
import LoginScreen from './src/screens/LoginScreen';
import ProductsScreen from './src/screens/ProductsScreen';
import styles from './src/styles/styles';

function Conteudo() {
  const { usuario, carregando } = useAuth();
  if (carregando) {
    return (
      <View style={styles.centro}>
        <ActivityIndicator size="large" />
        <Text style={styles.texto}>Verificando sessão...</Text>
      </View>
    );
  }
  if (!usuario) return <LoginScreen />;

  return (
    <SQLiteProvider databaseName="meucadastrodeprodutos.db" onInit={initializeDatabase}>
      <ProductsScreen key={usuario.uid} />
    </SQLiteProvider>
  );
}

export default function App() {
  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <AuthProvider>
          <Conteudo />
        </AuthProvider>
      </SafeAreaView>
    </SafeAreaProvider>
  );
}
```

Enquanto o Firebase verifica a sessão, aparece um indicador. Sem usuário, aparece o login. Com usuário, aparece a área interna. Ao sair, a tela de produtos é desmontada e o login reaparece; a senha digitada anteriormente não permanece no formulário. A chave `usuario.uid` reinicia o estado da tela quando a conta muda.

> Este exemplo controla o acesso pela interface e separa os produtos no SQLite pelo `uid`. Authentication não criptografa nem protege automaticamente um banco local contra acesso direto ao dispositivo. Se os produtos forem armazenados no Firebase futuramente, será necessário configurar as Security Rules do serviço escolhido.

## 13. Executar e testar

Na pasta `MeuCadastroDeProdutos`, execute:

```bash
npx expo start --clear
```

Abra no Expo Go pelo QR code. Para criar uma conta ou entrar, o dispositivo precisa de acesso à internet.

1. Confirme que o primeiro acesso mostra apenas o login.
2. Preencha e-mail e senha e toque em **Criar conta**. A área interna deve abrir.
3. No console, abra **Authentication > Users** e confirme a conta criada.
4. Cadastre `Caderno`, preço `12,50`, quantidade `10`.
5. Toque em **Alterar**, mude o preço para `15,00` e salve. Confirme que há apenas um registro, com o novo preço.
6. Teste nome vazio, preço negativo e quantidade fracionária: o formulário deve recusar esses dados.
7. Feche e abra o app. A sessão e os produtos devem permanecer.
8. Toque em **Sair** e confirme o retorno ao login.
9. Tente entrar com uma senha errada e confira a mensagem. Entre novamente com a senha correta.
10. Saia e crie uma segunda conta: a lista deve estar vazia. Volte à primeira conta e confira seus produtos.
11. Altere uma cor em `src/styles/styles.js` e observe a mudança nas telas.

O cadastro de contas fica no Firebase; os produtos persistem apenas naquele dispositivo. Desinstalar o app ou limpar seus dados pode apagar o SQLite, mas não remove a conta do Firebase.

## 14. Problemas comuns

| Problema | O que verificar |
|---|---|
| `auth/operation-not-allowed` | Ative o provedor E-mail/senha no console. |
| Chave inválida ou configuração ausente | Copie os valores corretos para `.env.local` e reinicie o Expo. |
| E-mail já cadastrado | Use **Entrar** com essa conta em vez de **Criar conta**. |
| Senha recusada | Confira os requisitos da política de senha configurada no Firebase. |
| Erro de rede | Confira a conexão do aparelho. |
| Erro ao importar Firebase | Instale com `npx expo install firebase` e consulte a compatibilidade no guia do Expo. |
| Produtos não aparecem em outro aparelho | O SQLite é local; este tutorial não sincroniza produtos. |
| Estilos não carregam | Confira o caminho do import e o `export default` em `styles.js`. |

## Referências

- [Firebase no Expo](https://docs.expo.dev/guides/using-firebase/)
- [Autenticação Firebase com e-mail e senha](https://firebase.google.com/docs/auth/web/password-auth)
- [Persistência de autenticação](https://firebase.google.com/docs/auth/web/auth-state-persistence)
- [SQLite no Expo](https://docs.expo.dev/versions/latest/sdk/sqlite/)
- [Estilos no React Native](https://reactnative.dev/docs/style)
