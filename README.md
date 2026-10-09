# 📱 Tutoriais de React Native para sala de aula

Este repositório reúne uma sequência de tutoriais práticos para o ensino de desenvolvimento mobile com **React Native** e **Expo**. O material foi organizado para acompanhar a evolução dos estudantes: da configuração do ambiente até a criação de aplicativos com navegação, persistência local, operações CRUD e autenticação.

Cada tutorial apresenta explicações, exemplos de código, atividades de verificação e desafios. Os projetos são construídos gradualmente para que cada novo conceito possa ser testado antes da próxima etapa.

## 🎯 Para quem é este material

- estudantes iniciantes em React Native;
- professores que desejam usar roteiros práticos em aula;
- pessoas que já conhecem lógica de programação e querem começar no desenvolvimento mobile;
- turmas que utilizam Android, iOS ou Expo Go para acompanhar as atividades.

## 📚 Sequência de tutoriais

Recomenda-se seguir os materiais na ordem abaixo:

| Aula | Tutorial | Principais conteúdos |
|---:|---|---|
| 1 | [Instalação e configuração do ambiente](1-tutorial_instalacao_configuracao.md) | Node.js, VS Code, Expo, Expo Go, primeiro projeto e GitHub |
| 2 | [Primeiro aplicativo Expo](2-tutorial_primeiro_projeto_expo.md) | Componentes básicos, `StyleSheet`, `TextInput`, `useState`, eventos, imagens e listas |
| 3 | [Estado, estilos e navegação](3-tutorial_projeto_expo_incremental_com_navegacao.md) | Aplicativo de hábitos, estado, conteúdo condicional, organização em telas e navegação inferior |
| 4 | [Navegação e SQLite](4-tutorial_projeto_expo_incremental_com_sqlite.md) | Aplicativo de produtos, banco local, pesquisa e CRUD completo |
| 5 | [Novos campos e cadastro de usuários](5-tutorial_incluir_novos_campos_e_cadastro_usuario.md) | Evolução do banco, novos campos de produtos e CRUD de usuários |
| 6 | [Autenticação com Firebase](6-tutorial_autenticacao_firebase_email_senha.md) | Novo app com login no Firebase, menu lateral (drawer), estilos externos e credenciais fora do Git |

## 🧭 Trilha de aprendizagem

Ao concluir a sequência, o estudante terá praticado:

1. preparação do ambiente de desenvolvimento;
2. criação e execução de projetos com Expo;
3. componentes, propriedades, estilos e layouts;
4. entrada de dados, eventos e gerenciamento de estado;
5. organização do aplicativo em arquivos e telas;
6. navegação entre páginas;
7. armazenamento local com SQLite;
8. cadastro, consulta, alteração e exclusão de dados;
9. autenticação com e-mail e senha usando Firebase.

## 🛠️ Pré-requisitos

Para começar, tenha disponível:

- **Node.js LTS** compatível com a versão atual do Expo;
- **npm**, instalado junto com o Node.js;
- **Visual Studio Code** ou outro editor de sua preferência;
- **Expo Go** em um dispositivo móvel ou um emulador Android/iOS;
- conhecimentos básicos de lógica de programação e JavaScript.

O primeiro tutorial orienta a instalação e a verificação dessas ferramentas. Os comandos usam a CLI local do Expo por meio de `npx`, portanto não é necessário instalar a Expo CLI globalmente.

## ▶️ Como usar em aula

1. Abra o tutorial correspondente à aula.
2. Execute as etapas na ordem apresentada.
3. Salve e teste o aplicativo depois de cada alteração.
4. Discuta o resultado antes de avançar para o próximo conceito.
5. Use os desafios finais como exercício individual ou atividade avaliativa.

Os tutoriais 3 a 6 formam uma progressão. O tutorial 5 parte do aplicativo desenvolvido no tutorial 4. O tutorial 6 começa um aplicativo novo, com autenticação Firebase.

## 📂 Organização do repositório

```text
.
├── 1-tutorial_instalacao_configuracao.md
├── 2-tutorial_primeiro_projeto_expo.md
├── 3-tutorial_projeto_expo_incremental_com_navegacao.md
├── 4-tutorial_projeto_expo_incremental_com_sqlite.md
├── 5-tutorial_incluir_novos_campos_e_cadastro_usuario.md
├── 6-tutorial_autenticacao_firebase_email_senha.md
└── README.md
```

Este repositório contém os **roteiros das aulas**. Durante cada tutorial, os estudantes criam os projetos e arquivos necessários seguindo as instruções apresentadas.

## 🔗 Documentação oficial

- [React Native](https://reactnative.dev/docs/getting-started)
- [Expo](https://docs.expo.dev/)
- [React Navigation](https://reactnavigation.org/docs/getting-started)
- [Expo SQLite](https://docs.expo.dev/versions/latest/sdk/sqlite/)
- [Firebase Authentication](https://firebase.google.com/docs/auth/)

## 🤝 Contribuições

Correções, sugestões de atividades e melhorias didáticas são bem-vindas. Ao propor uma alteração, procure manter a abordagem incremental e explicar os conceitos antes de apresentar soluções completas.

## 📄 Uso

Material produzido para fins educacionais e para apoio a aulas de React Native.
