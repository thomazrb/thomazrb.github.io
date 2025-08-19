---
title: "Criando um App com Flutter e Firebase"
date: 2025-08-19T11:03:03-03:00
draft: false
tags: ["Flutter", "Firebase", "Firestore", "Database"]
categories: ["Flutter"]
---

O poder do Flutter está em sua capacidade de criar aplicações para múltiplas plataformas a partir de uma única base de código. Quando combinado com o Firebase, esse poder se estende para a criação de apps conectados à nuvem de forma rápida e eficiente.

Neste guia, vamos demonstrar essa versatilidade construindo um aplicativo de cadastro de usuários (nome e CPF). Embora o foco do nosso exemplo seja a compilação para **Windows**, os mesmos princípios se aplicam para web, mobile e outras plataformas desktop, com pouquíssimas alterações.

### Pré-requisitos

Antes de começar, garanta que você tenha o ambiente configurado:

1.  **O SDK do Flutter** instalado e configurado no seu PATH.
2.  **Suporte para desenvolvimento Windows ativado no Flutter.** Você pode verificar se está tudo certo com o comando `flutter doctor`. Se algo estiver faltando na seção "Windows", siga as instruções fornecidas pelo próprio comando.
3.  **Uma conta Google** para acessar o Firebase.
4.  **Node.js e npm** instalados. Eles são necessários para a CLI do Firebase. Baixe a versão **LTS** no site oficial [nodejs.org](https://nodejs.org/).

### Passo 1: Preparando o Ambiente Firebase

Este passo prepara seu computador para se comunicar com o Firebase. Se você já fez isso antes, pode pular para o próximo passo.

1.  **Instale a CLI globalmente** via npm. Isso adiciona os comandos do Firebase ao seu sistema:
    ```bash
    npm install -g firebase-tools
    ```

2.  **Faça login** na sua conta Google para autenticar sua máquina:
    ```bash
    firebase login
    ```
    Este comando abrirá uma janela no seu navegador para autenticação.

### Passo 2: Configurar o Projeto no Console do Firebase

1.  Acesse o [Console do Firebase](https://console.firebase.google.com/).
2.  Clique em **"Adicionar projeto"** e dê um nome a ele (ex: `flutter-firebase-db-example`).
3.  Siga os passos de configuração. Não é necessário ativar o Google Analytics.
4.  No menu à esquerda, vá em **Build > Firestore Database** e clique em **"Criar banco de dados"**.
5.  Selecione **"Iniciar em modo de teste"**.
    * **Aviso:** O modo de teste é aberto para leitura e escrita. Para um app real, configure regras de segurança mais restritivas.
6.  Escolha uma localização para os dados e clique em **"Ativar"**.

### Passo 3: Criando o Projeto Flutter

Com o ambiente pronto, vamos criar nosso projeto Flutter.

1.  Abra seu terminal ou prompt de comando.
2.  Execute o comando abaixo para criar o projeto:
    ```bash
    flutter create flutter_firebase_db_example
    ```
3.  Navegue para dentro da pasta do projeto recém-criado. Todos os próximos comandos devem ser executados a partir daqui.
    ```bash
    cd flutter_firebase_db_example
    ```

### Passo 4: Adicionar as Dependências no Flutter

Agora, vamos adicionar os pacotes para que nosso app Flutter possa se comunicar com o Firebase. A forma mais rápida e segura de fazer isso é usando o terminal.

1.  Na raiz do seu projeto, execute o comando abaixo para adicionar os pacotes `firebase_core` e `cloud_firestore` de uma só vez:
    ```bash
    flutter pub add firebase_core cloud_firestore
    ```
    Este comando automaticamente encontrará as versões mais recentes dos pacotes, as adicionará ao seu arquivo `pubspec.yaml` e executará o `flutter pub get` por você.

> **Solucionando Problemas no Windows:** Se você encontrar um erro como `ERROR_INVALID_FUNCTION` ao rodar o comando acima, é muito provável que seu projeto Flutter esteja em um drive diferente do seu SDK (por exemplo, projeto no drive `D:` e SDK no `C:`). Para resolver, mova a pasta do seu projeto para o mesmo drive do SDK do Flutter (geralmente `C:`) e tente novamente.

### Passo 5: Conectar o App ao Firebase com FlutterFire

A CLI do FlutterFire automatiza a conexão do seu projeto com o Firebase.

1.  Instale a CLI do FlutterFire (se ainda não tiver):
    ```bash
    dart pub global activate flutterfire_cli
    ```
    > **Atenção ao "Warning" no Windows:** Após executar o comando acima, você pode ver um aviso dizendo que o diretório `...Pub\Cache\bin` não está no seu "Path". **Este aviso é importante!** Significa que o terminal não encontrará o comando `flutterfire` no próximo passo.
    >
    > **Como resolver:**
    > 1.  Pesquise por "Variáveis de Ambiente" no menu Iniciar do Windows e abra a opção "Editar as variáveis de ambiente do sistema".
    > 2.  Na janela que abrir, clique em "Variáveis de Ambiente...".
    > 3.  Na seção "Variáveis de usuário", encontre e selecione a variável `Path` e clique em "Editar...".
    > 4.  Clique em "Novo" e cole o caminho que apareceu no seu aviso (geralmente `C:\Users\SEU_USUARIO\AppData\Local\Pub\Cache\bin`).
    > 5.  Clique em "OK" em todas as janelas para salvar.
    > 6.  **Feche e abra novamente seu terminal** para que as alterações tenham efeito.

2.  Na raiz do seu projeto, execute o comando de configuração:
    ```bash
    flutterfire configure
    ```
    O comando irá detectar seu projeto Firebase e perguntar para quais plataformas você quer configurar. **Certifique-se de selecionar a opção `windows`**. Ele então irá gerar o arquivo `lib/firebase_options.dart` com as chaves corretas para a sua aplicação desktop.
    > **Dica:** Você não precisa selecionar todas as plataformas de uma vez. A melhor prática é selecionar apenas as que você está desenvolvendo no momento (como `windows` para este guia). Se no futuro você decidir compilar seu app para Android, iOS ou web, basta rodar o comando `flutterfire configure` novamente e adicionar a nova plataforma. A escolha não é permanente!

### Passo 6: O Código do Aplicativo

Agora, a parte divertida! Substitua todo o conteúdo do seu arquivo `lib/main.dart` pelo código completo abaixo. Ele cria a interface para listar e adicionar usuários.

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'firebase_options.dart'; // Importa as configurações geradas pelo FlutterFire

// Ponto de entrada principal da aplicação
void main() async {
  // Garante que os bindings do Flutter foram inicializados
  WidgetsFlutterBinding.ensureInitialized();
  // Inicializa o Firebase usando as opções da plataforma atual (Windows, neste caso)
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter App Desktop com Firestore',
      theme: ThemeData(
        primarySwatch: Colors.indigo,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: const HomePage(),
      debugShowCheckedModeBanner: false,
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  // Controllers para os campos de texto do diálogo
  final TextEditingController _nomeController = TextEditingController();
  final TextEditingController _cpfController = TextEditingController();

  // Referência para a coleção 'usuarios' no Firestore
  final CollectionReference _usuarios = FirebaseFirestore.instance.collection(
    'usuarios',
  );

  // Função para exibir o diálogo de adicionar usuário
  Future<void> _mostrarDialogoAdicionar() async {
    // Limpa os controllers antes de abrir o diálogo
    _nomeController.clear();
    _cpfController.clear();

    await showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: const Text('Adicionar Novo Usuário'),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                controller: _nomeController,
                decoration: const InputDecoration(labelText: 'Nome'),
              ),
              TextField(
                controller: _cpfController,
                decoration: const InputDecoration(labelText: 'CPF'),
              ),
            ],
          ),
          actions: [
            TextButton(
              child: const Text('Cancelar'),
              onPressed: () {
                Navigator.of(context).pop();
              },
            ),
            ElevatedButton(
              child: const Text('Adicionar'),
              onPressed: () {
                final String nome = _nomeController.text;
                final String cpf = _cpfController.text;
                if (nome.isNotEmpty && cpf.isNotEmpty) {
                  // Adiciona um novo documento à coleção 'usuarios'
                  _usuarios.add({"nome": nome, "cpf": cpf});

                  // Fecha o diálogo
                  Navigator.of(context).pop();
                }
              },
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Cadastro de Usuários')),
      // StreamBuilder escuta as mudanças na coleção do Firestore em tempo real
      body: StreamBuilder(
        stream: _usuarios.snapshots(), // O stream que será ouvido
        builder: (context, AsyncSnapshot<QuerySnapshot> streamSnapshot) {
          if (streamSnapshot.hasData) {
            // Se houver dados, constrói uma ListView
            return ListView.builder(
              itemCount: streamSnapshot.data!.docs.length,
              itemBuilder: (context, index) {
                final DocumentSnapshot documentSnapshot =
                    streamSnapshot.data!.docs[index];
                return Card(
                  margin: const EdgeInsets.all(10),
                  child: ListTile(
                    title: Text(documentSnapshot['nome']),
                    subtitle: Text(documentSnapshot['cpf']),
                  ),
                );
              },
            );
          }
          // Enquanto os dados estão carregando, mostra um indicador de progresso
          return const Center(child: CircularProgressIndicator());
        },
      ),
      // Botão flutuante para adicionar novos usuários
      floatingActionButton: FloatingActionButton(
        onPressed: () => _mostrarDialogoAdicionar(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```
*Nota: O código-fonte completo deste projeto está disponível no GitHub: [flutter_firebase_db_example](https://github.com/thomazrb/flutter_firebase_db_example).*

### Passo 7: Testar e Gerar o Execútavel

Com o código pronto, vamos rodar e compilar nosso aplicativo para Windows.

1.  **Teste em modo de depuração:**
    ```bash
    flutter run -d windows
    ```
    Uma janela nativa do Windows abrirá com seu aplicativo. Tente adicionar alguns usuários e veja a mágica do tempo real acontecer.

2.  **Gere o executável (`.exe`):**
    ```bash
    flutter build windows
    ```
    Após a conclusão, o Flutter informará onde encontrar o executável. Geralmente, ele estará na pasta `build\windows\runner\Release`. Você pode pegar todo o conteúdo dessa pasta, compactá-lo e distribuí-lo para outros computadores com Windows!

### Conclusão

Parabéns! Você criou um aplicativo desktop nativo para Windows com uma base de dados na nuvem, usando uma única base de código com Flutter. Isso demonstra a flexibilidade incrível da ferramenta, permitindo que você leve suas ideias para praticamente qualquer plataforma sem reescrever tudo do zero.