---
title: "Autenticação com Flutter e Firebase: Guia de Login com Email e Senha"
date: 2025-09-24T16:55:38-03:00
draft: false
tags: ["Flutter", "Firebase", "Authentication", "Login"]
categories: ["Flutter"]
---

A autenticação é a porta de entrada para a maioria dos aplicativos modernos. Proteger rotas, personalizar a experiência do usuário e garantir a segurança dos dados são tarefas fundamentais. Felizmente, o Firebase Authentication simplifica drasticamente esse processo.

Neste guia, vamos construir um aplicativo Flutter simples com uma tela de login (usando email e senha) e uma tela inicial que só pode ser acessada após a autenticação. O foco é criar uma base sólida e fácil de entender para que você possa implementar em seus próprios projetos.

### Pré-requisitos

O ambiente necessário é muito parecido com o de outros projetos Flutter/Firebase:

1.  **O SDK do Flutter** instalado e configurado no seu PATH.
2.  **Suporte para a plataforma desejada ativado** (Android, iOS, Web, Windows, etc.). Verifique com o comando `flutter doctor`.
3.  **Uma conta Google** para acessar o Firebase.
4.  **Node.js e npm** instalados (necessários para a CLI do Firebase).

### Passo 1: Preparando o Ambiente Firebase

Se você já usa a CLI do Firebase, pode pular esta etapa.

1.  **Instale a CLI globalmente** via npm:
    ```bash
    npm install -g firebase-tools
    ```

2.  **Faça login** na sua conta Google para autenticar sua máquina:
    ```bash
    firebase login
    ```
    Isso abrirá uma janela no seu navegador para você completar o login.

### Passo 2: Configurar o Projeto no Console do Firebase

1.  Acesse o [Console do Firebase](https://console.firebase.google.com/).
2.  Clique em **"Adicionar projeto"** e dê um nome a ele (ex: `flutter-auth-example`).
3.  No menu à esquerda, vá em **Build > Authentication**. Esta é a seção do Firebase dedicada ao gerenciamento de usuários. Ela oferece um serviço de backend completo, SDKs fáceis de usar e bibliotecas de UI prontas para autenticar usuários no seu aplicativo. Com isso, você não precisa se preocupar em criar e manter seu próprio servidor de autenticação.
4.  Clique no botão **"Get started"**.
5.  Na aba **"Sign-in method"**, você verá uma lista de provedores de autenticação. O Firebase é extremamente flexível, oferecendo métodos como login com contas Google, Facebook, Apple, GitHub, além de opções como login anônimo ou por número de telefone. Cada um serve a um propósito diferente. Para manter este guia focado e simples, **selecione "E-mail/senha"**, que é o método de login mais tradicional e uma ótima base para começar.
6.  **Ative** a opção e clique em **"Salvar"**.
7.  Ainda na aba de Autenticação, vá para a aba **"Users"** e clique em **"Add user"**. Crie um usuário de teste com um email e senha para que possamos testar nosso login mais tarde.

### Passo 3: Criando o Projeto Flutter

Vamos criar um novo projeto Flutter do zero.

1.  Abra seu terminal e execute o comando:
    ```bash
    flutter create flutter_auth_example
    ```

2.  Navegue para dentro da pasta do projeto:
    ```bash
    cd flutter_auth_example
    ```

### Passo 4: Adicionar as Dependências no Flutter

Precisamos de dois pacotes principais para este projeto: o `firebase_core` para inicializar a conexão e o `firebase_auth` para cuidar da autenticação.

1.  Na raiz do seu projeto, execute o comando:
    ```bash
    flutter pub add firebase_core firebase_auth
    ```

### Passo 5: Conectar o App ao Firebase com FlutterFire

A CLI do FlutterFire é a ferramenta que conecta seu código Flutter ao projeto que criamos no console do Firebase.

1.  Se ainda não tiver, instale a CLI do FlutterFire:
    ```bash
    dart pub global activate flutterfire_cli
    ```
    > **Atenção:** Se o terminal avisar que o diretório `Pub\Cache\bin` não está no seu "Path", siga as instruções do aviso para adicioná-lo às suas variáveis de ambiente e **reinicie o terminal**.

2.  Na raiz do seu projeto, execute o comando de configuração:
    ```bash
    flutterfire configure
    ```
    A ferramenta irá listar seus projetos Firebase. Selecione o `flutter-auth-example` que criamos. Em seguida, escolha as plataformas para as quais você quer configurar o app (ex: android, ios, web). Ao final, o arquivo `lib/firebase_options.dart` será gerado automaticamente.

### Passo 6: O Código do Aplicativo

Agora, vamos ao código! Para uma melhor organização, vamos dividir nossa lógica em três arquivos. Crie uma nova pasta `pages` dentro da sua pasta `lib`.

**1. Crie o arquivo `lib/pages/login_page.dart`**
Este arquivo conterá exclusivamente o widget da nossa tela de login. O `StatefulWidget` é usado aqui porque precisamos gerenciar o estado do formulário (o que o usuário digita) e o estado de carregamento (`_isLoading`).

```dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';

// Tela de Login
class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  bool _isLoading = false;

  // Função para lidar com o processo de login
  Future<void> _login() async {
    setState(() {
      _isLoading = true;
    });

    try {
      // Tenta fazer login com email e senha
      await FirebaseAuth.instance.signInWithEmailAndPassword(
        email: _emailController.text.trim(),
        password: _passwordController.text.trim(),
      );
      // Se o login for bem-sucedido, o StreamBuilder no AuthWrapper
      // irá reconstruir e levar o usuário para a HomePage.
    } on FirebaseAuthException catch (e) {
      // Mostra um erro para o usuário se o login falhar
      final snackBar = SnackBar(
        content: Text('Erro ao fazer login: ${e.message}'),
        backgroundColor: Colors.red,
      );
      ScaffoldMessenger.of(context).showSnackBar(snackBar);
    } finally {
      // Garante que o estado de loading seja desativado
      if (mounted) {
        setState(() {
          _isLoading = false;
        });
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Center(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.lock, size: 80, color: Colors.blue),
              const SizedBox(height: 20),
              TextField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.email),
                ),
                keyboardType: TextInputType.emailAddress,
              ),
              const SizedBox(height: 16),
              TextField(
                controller: _passwordController,
                decoration: const InputDecoration(
                  labelText: 'Senha',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.vpn_key),
                ),
                obscureText: true,
              ),
              const SizedBox(height: 24),
              // Mostra o botão ou o loader
              _isLoading
                  ? const CircularProgressIndicator()
                  : ElevatedButton(
                      onPressed: _login,
                      style: ElevatedButton.styleFrom(
                        padding: const EdgeInsets.symmetric(
                          horizontal: 50,
                          vertical: 15,
                        ),
                      ),
                      child: const Text('Entrar'),
                    ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**2. Crie o arquivo `lib/pages/home_page.dart`**
Este arquivo conterá o widget da tela principal. Ele é um `StatelessWidget` porque seu único trabalho é exibir informações (o email do usuário) que ele recebe de fora, sem gerenciar nenhum estado interno complexo.

```dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';

// Tela Home, exibida após o login
class HomePage extends StatelessWidget {
  final User user;

  const HomePage({super.key, required this.user});

  // Função para fazer logout
  Future<void> _logout() async {
    await FirebaseAuth.instance.signOut();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Página Inicial'),
        actions: [
          IconButton(
            icon: const Icon(Icons.logout),
            onPressed: _logout,
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Bem-vindo!',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            Text(
              user.email ?? 'Email não disponível',
              style: const TextStyle(fontSize: 18),
            ),
          ],
        ),
      ),
    );
  }
}
```

**3. Substitua o conteúdo do `lib/main.dart`**
Agora, nosso arquivo principal fica mais limpo. Sua principal responsabilidade é inicializar o Firebase e usar o `AuthWrapper` para decidir qual tela (Login ou Home) deve ser mostrada ao usuário com base em seu status de autenticação.

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'firebase_options.dart'; // Importa as configurações
import 'pages/home_page.dart'; // Importa a HomePage
import 'pages/login_page.dart'; // Importa a LoginPage

// Ponto de entrada da aplicação
void main() async {
  // Garante que os widgets do Flutter estão prontos
  WidgetsFlutterBinding.ensureInitialized();
  // Inicializa o Firebase
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Firebase Auth',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      // O AuthWrapper decide qual tela mostrar
      home: const AuthWrapper(),
      debugShowCheckedModeBanner: false,
    );
  }
}

// O AuthWrapper ouve as mudanças de estado de autenticação
class AuthWrapper extends StatelessWidget {
  const AuthWrapper({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      // Ouve o stream de estado de autenticação do Firebase
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snapshot) {
        // Enquanto está conectando, mostra um loader
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }
        // Se o snapshot tem dados, significa que o usuário está logado
        if (snapshot.hasData) {
          return HomePage(user: snapshot.data!);
        }
        // Se não tem dados, mostra a tela de login
        return const LoginPage();
      },
    );
  }
}
```

### Passo 7: Testando a Aplicação

Com o código no lugar e o usuário de teste criado no console do Firebase, é hora de testar.

1.  **Execute o aplicativo** na sua plataforma de preferência (Android, iOS, Web, etc.):
    ```bash
    flutter run
    ```

2.  A tela de login deve aparecer.
3.  Use o **email e a senha** do usuário que você cadastrou no Passo 2.
4.  Ao clicar em "Entrar", você deve ser redirecionado para a tela inicial, que exibirá seu email.
5.  Clique no ícone de "logout" na barra de aplicativos. Você será levado de volta para a tela de login.
6.  Tente fazer login com uma senha errada para ver a mensagem de erro aparecer na parte inferior da tela.

### Conclusão

Parabéns! Você implementou um fluxo de autenticação completo e seguro com Flutter e Firebase. A estrutura que criamos, usando um `StreamBuilder` para "ouvir" o estado de login e separando as telas em arquivos diferentes, é uma das formas mais robustas e eficientes de gerenciar sessões de usuário em um aplicativo Flutter. A partir daqui, você pode expandir facilmente para incluir uma tela de registro, funcionalidade de "esqueci minha senha" ou login com provedores sociais como Google e Facebook.