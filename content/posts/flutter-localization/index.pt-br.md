---
title: "Tutorial: Internacionalização de Aplicativos Flutter com 'flutter_localizations' e 'intl'"
date: 2025-06-30T01:36:52-03:00
draft: false
tags: ["Flutter", "Localization"]
categories: ["Flutter"]
---

Este tutorial irá guiá-lo através do processo de adicionar suporte a múltiplos idiomas (internacionalização e localização) ao seu aplicativo Flutter, utilizando os pacotes `flutter_localizations` e `intl` para widgets do Material Design.

## 1. Introdução

A **internacionalização (i18n)** é o processo de projetar e desenvolver um aplicativo para que ele possa ser adaptado a diferentes idiomas e regiões sem alterações de engenharia. A **localização (l10n)** é o processo de adaptar um aplicativo para um local ou mercado específico, adicionando componentes específicos do local e traduzindo o texto.

No Flutter, o pacote **`flutter_localizations`** fornece as localizações para widgets padrão do Material Design, enquanto o pacote **`intl`** é a biblioteca Dart que lida com formatação de mensagens, datas, números e texto bidirecional.

## 2. Pré-requisitos

* Flutter SDK instalado e configurado.

* Um projeto Flutter existente.

* Conhecimento básico de desenvolvimento Flutter e Dart.

## 3. Passo a Passo

### Passo 1: Instalar as Dependências

Você pode editar o `pubspec.yaml` manualmente, ou usar o comando `flutter pub add` para adicionar os pacotes. Optei pela segunda opção. Abra o terminal na raiz do seu projeto Flutter e execute os seguintes comandos:

```bash
flutter pub add flutter_localizations --sdk=flutter
flutter pub add intl
```

* O comando `flutter pub add flutter_localizations --sdk=flutter` adiciona `flutter_localizations` especificando que ele faz parte do SDK do Flutter, ele não é uma biblioteca de terceiros.

* O comando `flutter pub add intl` adiciona o pacote `intl` como uma dependência normal.

Após executar esses comandos, seu arquivo `pubspec.yaml` será atualizado automaticamente, e os pacotes serão baixados. Você deverá ver algo como:

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: ^0.20.2 # A versão pode ser diferente
```

### Passo 2: Habilitar a Geração de Código de Localização

Ainda no `pubspec.yaml`, adicione a flag `generate: true` na seção `flutter`. Isso instrui o Flutter a gerar automaticamente os arquivos de localização a partir dos seus arquivos ARB (Application Resource Bundle).

```yaml
flutter:
  uses-material-design: true
  generate: true # Adicione esta linha para habilitar a geração de código de localização
  # ... outras configurações do Flutter
```

Execute `flutter pub get` no seu terminal após esta alteração para garantir que as configurações sejam aplicadas.

### Passo 3: Criar o Arquivo de Configuração `l10n.yaml`

Crie um novo arquivo chamado **`l10n.yaml`** na raiz do seu projeto (no mesmo nível do `pubspec.yaml`). Este arquivo configura como o Flutter deve gerar os arquivos de localização.

```yaml
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
```

* `arb-dir`: O diretório onde seus arquivos `.arb` (com as traduções) serão armazenados. Vamos criá-lo em `lib/l10n`.

* `template-arb-file`: O nome do seu arquivo ARB base (geralmente o idioma padrão, como inglês).

* `output-localization-file`: O nome do arquivo Dart que será gerado com as classes de localização.

### Passo 4: Criar os Arquivos ARB (Application Resource Bundle)

Crie o diretório especificado em `arb-dir`, que é **`lib/l10n`**. Dentro dele, crie seus arquivos ARB para cada idioma que você deseja suportar.

**`lib/l10n/app_en.arb` (Inglês - arquivo base):**

```json
{
  "@@locale": "en",
  "helloWorld": "Hello World!",
  "@helloWorld": {
    "description": "A conventional greeting"
  },
  "welcomeMessage": "Welcome, {name}!",
  "@welcomeMessage": {
    "description": "A welcome message with a name parameter",
    "placeholders": {
      "name": {
        "type": "String"
      }
    }
  }
}
```

**`lib/l10n/app_pt.arb` (Português):**

```json
{
  "@@locale": "pt",
  "helloWorld": "Olá Mundo!",
  "welcomeMessage": "Bem-vindo(a), {name}!"
}
```

**Explicação dos arquivos ARB:**

* `"@@locale": "en"`: Define o idioma para este arquivo.

* `"helloWorld": "Hello World!"`: A chave da sua string e sua tradução.

* `"@helloWorld"`: Metadados opcionais para a chave, como uma descrição para tradutores.

* `"welcomeMessage": "Welcome, {name}!"`: Exemplo de string com um placeholder (`{name}`). O `@welcomeMessage` define o tipo do placeholder (esses são obrigatórios quando a string contêm parâmetros para que o gerador de código do Flutter saiba como criar a função Dart correspondente).
### Passo 5: Gerar os Arquivos de Localização

Após criar ou modificar seus arquivos ARB, você precisa gerar o código Dart correspondente. Execute o seguinte comando no terminal na raiz do seu projeto:

```bash
flutter gen-l10n
```

Este comando lerá seus arquivos `.arb` e o `l10n.yaml` e gerará o arquivo `app_localizations.dart` (e outros arquivos necessários) no local configurado. Se o arquivo já existe, ele será atualizado.

### Passo 6: Configurar o `MaterialApp`

No seu arquivo `main.dart` (ou onde você define seu `MaterialApp`), importe os arquivos de localização gerados e configure as propriedades `localizationsDelegates` e `supportedLocales`.

**`lib/main.dart` (exemplo):**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
// ATENÇÃO: O import abaixo assume que 'app_localizations.dart' é gerado diretamente em 'lib/l10n/'.
// Embora a abordagem padrão do Flutter seja
// 'package:flutter_gen/gen_l10n/app_localizations.dart;',
// se o seu ambiente gera o arquivo neste local, use esta linha.
import 'l10n/app_localizations.dart';


void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Localization Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      // Propriedades de localização
      localizationsDelegates: const [
        AppLocalizations.delegate, // Delegate gerado pelos seus arquivos ARB
        GlobalMaterialLocalizations.delegate, // Localizações para widgets Material
        GlobalWidgetsLocalizations.delegate,  // Localizações para widgets básicos
      ],
      supportedLocales: const [
        Locale('en'), // Inglês
        Locale('pt'), // Português
        // Adicione outros idiomas que você suporta
      ],
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    // Acessando as strings localizadas
    final appLocalizations = AppLocalizations.of(context)!;

    return Scaffold(
      appBar: AppBar(
        title: Text(appLocalizations.helloWorld),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              appLocalizations.welcomeMessage('Usuário'), // Usando a string com placeholder
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            const SizedBox(height: 20),
            Text(
              'Idioma atual: ${Localizations.localeOf(context).languageCode}',
              style: Theme.of(context).textTheme.bodyLarge,
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explicação:**

* `import 'l10n/app_localizations.dart';`: Este é o caminho de importação que você solicitou, assumindo que `app_localizations.dart` é gerado diretamente na pasta `lib/l10n/`. Observe que a abordagem padrão do Flutter para arquivos gerados é `package:flutter_gen/gen_l10n/app_localizations.dart;`. Escolha o caminho que funciona para o seu ambiente.

* `AppLocalizations.delegate`: É o delegado específico para seu aplicativo e que sabe como carregar as traduções dos seus arquivos ARB.

No Flutter, um `LocalizationsDelegate` é uma classe que sabe como carregar recursos específicos de um Locale (idioma e região) para o seu aplicativo. É como um "gerente" que sabe onde encontrar as traduções e como disponibilizá-las para os seus widgets.

* `GlobalMaterialLocalizations.delegate`, `GloalWidgetsLocalizations.delegate`: São os delegados fornecidos pelo `flutter_localizations` (ou seja, é nativo do próprio flutter SDK) que fornecem traduções para os widgets padrão do Material Design (como `DatePicker`, `TimePicker`, botões de navegação, etc.), fazendo com que nomes de meses, botões de ok, cancelar, voltar, etc. sejam traduzidos.

* `supportedLocales`: Uma lista de todos os `Locale`s que seu aplicativo suporta. O Flutter tentará corresponder o idioma do dispositivo do usuário com um dos idiomas nesta lista.

### Passo 7: Usar as Strings Localizadas no Seu Código

Para acessar suas strings traduzidas, você usará a classe `AppLocalizations` (o nome da classe é derivado do `output-localization-file` no `l10n.yaml`).

```dart
// Dentro de um widget que tem acesso a um BuildContext
final appLocalizations = AppLocalizations.of(context)!;

// Para uma string simples:
Text(appLocalizations.helloWorld)

// Para uma string com placeholder:
Text(appLocalizations.welcomeMessage('NomeDoUsuario'))
```

Quando você executa seu aplicativo, o Flutter detectará o idioma do dispositivo e carregará as traduções correspondentes. Se o idioma do dispositivo não for suportado, ele usará o idioma padrão (definido no `template-arb-file` do `l10n.yaml`, que é `en` neste exemplo).

### Passo 8: Executar o Aplicativo

Salve todos os arquivos e execute seu aplicativo Flutter:

```bash
flutter run
```

Seu aplicativo agora deve exibir o texto "Olá Mundo!" ou "Hello World!" e a mensagem de boas-vindas de acordo com o idioma do seu dispositivo (se for português ou inglês).

### Dicas Adicionais:

* **Alterar o Idioma do Dispositivo:** Para testar a localização, você pode alterar o idioma do seu emulador ou dispositivo físico nas configurações do sistema.

* **Importação Correta:** Verifique sempre o caminho exato do arquivo `app_localizations.dart` que o Flutter gera em seu projeto. O caminho pode ser `import 'l10n/app_localizations.dart';` (se gerado diretamente em `lib/l10n/`) ou `import 'package:flutter_gen/gen_l10n/app_localizations.dart';` (o padrão mais comum). Certifique-se de usar o caminho que o seu ambiente Flutter está utilizando.

* **Adicionar Novos Idiomas:** Para adicionar um novo idioma, basta criar um novo arquivo `.arb` (ex: `app_es.arb` para espanhol) no diretório `lib/l10n` e adicionar o `Locale('es')` à lista `supportedLocales` no seu `MaterialApp`. **Lembre-se de rodar `flutter gen-l10n` novamente** para que o novo idioma e suas traduções sejam incluídos no código gerado.