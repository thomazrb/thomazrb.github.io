---
title: "Persistência de Dados no Flutter com SharedPreferences"
date: 2025-12-10T11:18:38-03:00
draft: false
tags: ["Flutter", "SharedPreferences", "Persistência", "Armazenamento Local", "Dart"]
categories: ["Flutter"]
---

Quando desenvolvemos aplicativos, uma necessidade comum é preservar dados simples entre sessões do usuário. O SharedPreferences é a solução ideal para armazenar pequenas quantidades de dados no formato chave-valor, como preferências do usuário, configurações da aplicação ou estados simples que precisam persistir.

Neste tutorial, vamos implementar persistência de dados no aplicativo contador padrão do Flutter, fazendo com que o valor do contador seja mantido mesmo após fechar e reabrir o aplicativo.

### O que é SharedPreferences?

SharedPreferences é uma API multiplataforma que permite armazenar dados primitivos de forma persistente no dispositivo. Funciona como um dicionário de chave-valor, onde você pode salvar e recuperar dados de tipos básicos como strings, inteiros, booleanos e listas de strings.

**Implementação por Plataforma:**

* **Android:** Utiliza o `SharedPreferences` nativo, armazenando dados em arquivos XML no diretório privado da aplicação.
* **iOS/macOS:** Utiliza o `NSUserDefaults`, armazenando dados em arquivos plist.
* **Web:** Utiliza `localStorage` do navegador. **Importante:** os dados são vinculados ao domínio e ao navegador específico. Se o usuário limpar o cache/cookies do navegador, os dados serão perdidos. Se acessar o mesmo site em outro navegador ou em modo anônimo, não terá acesso aos dados salvos anteriormente. Para aplicações web que precisam de persistência mais confiável, considere usar backend com autenticação.
* **Windows:** Armazena dados em arquivos JSON no diretório de configuração do usuário (tipicamente `C:\Users\{usuario}\AppData\Roaming\{nome_da_empresa}\{nome_do_app}`).
* **Linux:** Armazena dados em arquivos JSON no diretório de configuração do usuário (tipicamente `~/.local/share/{nome_do_app}` ou `~/.config/{nome_do_app}`).

**Quando Usar SharedPreferences:**

SharedPreferences é apropriado para:
* Preferências de configuração (tema, idioma, notificações)
* Estados simples da aplicação
* Dados de onboarding
* Pequenas quantidades de dados não-sensíveis

**Quando NÃO Usar:**

* Dados sensíveis (senhas, tokens) - para isso use o `flutter_secure_storage` que é mais seguro
* Grandes volumes de dados - use bancos de dados como SQLite
* Dados estruturados complexos - considere Hive ou ObjectBox
* Dados que exigem consultas complexas

### Passo 1: Adicionar a Dependência

Primeiro, adicione o pacote `shared_preferences` ao seu projeto. No terminal, na raiz do projeto, execute:

```bash
flutter pub add shared_preferences
```

Isso adiciona a linha necessária ao seu `pubspec.yaml` e baixa o pacote.

### Passo 2: Estrutura do Código

Vamos modificar o aplicativo contador padrão do Flutter (o template criado quando você inicia um novo projeto) para incluir persistência, ou seja, se você parar o programa e iniciar ele novamente, o valor do contador irá persistir. A lógica é simples:

1. Ao iniciar o app, carregar o valor salvo
2. Sempre que o contador for incrementado, salvar o novo valor
3. O valor permanecerá disponível mesmo após reiniciar o aplicativo

### Passo 3: Implementação Completa

Substitua o conteúdo do arquivo `lib/main.dart` pelo código abaixo:

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart'; // NOVO: importação do pacote

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  // NOVO: Chave para salvar/recuperar o valor do SharedPreferences
  static const String _counterKey = 'counter';

  // NOVO: Método initState para carregar o valor salvo quando o app inicia
  @override
  void initState() {
    super.initState();
    _loadCounter();
  }

  // NOVO: Carrega o valor do contador do SharedPreferences
  Future<void> _loadCounter() async {
    final prefs = await SharedPreferences.getInstance();
    setState(() {
      _counter = prefs.getInt(_counterKey) ?? 0;
    });
  }

  // NOVO: Salva o valor do contador no SharedPreferences
  Future<void> _saveCounter() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setInt(_counterKey, _counter);
  }

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
    _saveCounter(); // NOVO: salva o valor após incrementar
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Entendendo o Código

Vamos analisar apenas as partes que foram adicionadas ao código padrão do Flutter:

**1. Importação e Constante:**
```dart
import 'package:shared_preferences/shared_preferences.dart'; // NOVO
static const String _counterKey = 'counter'; // NOVO
```
Importamos o pacote e definimos uma constante para a chave. A **chave** funciona como um identificador único (similar a um nome de variável) que será usado para armazenar e recuperar o valor do contador. Por exemplo, quando salvamos `prefs.setInt('counter', 5)`, estamos dizendo "salve o valor 5 com o identificador 'counter'". Depois, quando quisermos recuperar esse valor, usamos o mesmo identificador: `prefs.getInt('counter')`. 

**Por que usar uma constante?** Embora seja possível escrever diretamente `'counter'` em cada lugar, usar uma constante (`_counterKey`) traz benefícios importantes: se você precisar mudar o nome da chave, basta alterar em um único lugar; evita erros de digitação (se você escrever `'conter'` por engano em algum lugar, o código não vai funcionar); e facilita manutenção em projetos maiores onde você pode ter dezenas de chaves diferentes.

**2. Método initState:**
```dart
@override
void initState() {
  super.initState();
  _loadCounter(); // NOVO
}
```
O `initState()` é chamado automaticamente quando o widget é criado. Neste caso, o widget é a página `MyHomePage`. Aqui carregamos o valor salvo assim que a página é inicializada.

**3. Carregamento dos Dados:**
```dart
Future<void> _loadCounter() async {
  final prefs = await SharedPreferences.getInstance();
  setState(() {
    _counter = prefs.getInt(_counterKey) ?? 0;
  });
}
```
Aqui usamos `Future` e `async` porque acessar o SharedPreferences envolve leitura de disco/armazenamento, que é uma operação **assíncrona** (ou seja, leva um tempo para completar). O `async/await` permite que o Flutter execute outras tarefas (como manter a interface responsiva) enquanto espera a operação terminar. Quando usamos `await`, estamos dizendo: "pause este método aqui e continue quando `getInstance()` terminar, mas não trave o resto do aplicativo". Obtemos a instância do SharedPreferences e carregamos o valor salvo. O operador `??` fornece `0` como valor padrão se a chave não existir.

**4. Salvamento dos Dados:**
```dart
Future<void> _saveCounter() async {
  final prefs = await SharedPreferences.getInstance();
  await prefs.setInt(_counterKey, _counter);
}
```
Assim como no carregamento, usamos `Future` e `async` porque gravar dados no armazenamento é uma operação assíncrona. O `async/await` permite que o aplicativo continue funcionando normalmente enquanto os dados são salvos em segundo plano. Salvamos o valor atual do contador. Esse método é chamado sempre que incrementamos o contador.

**5. Chamada no Incremento:**
```dart
void _incrementCounter() {
  setState(() {
    _counter++;
  });
  _saveCounter(); // NOVO: apenas esta linha foi adicionada
}
```
Após incrementar, salvamos o novo valor automaticamente.

**Métodos Disponíveis no SharedPreferences:**

O SharedPreferences oferece métodos para diferentes tipos de dados:
* `setInt()` / `getInt()` - números inteiros
* `setDouble()` / `getDouble()` - números decimais
* `setString()` / `getString()` - textos
* `setBool()` / `getBool()` - booleanos
* `setStringList()` / `getStringList()` - listas de strings

Todos os métodos `set` retornam `Future<bool>`, e todos os métodos `get` podem retornar `null` se a chave não existir.

### Considerações Importantes

**Performance:**
* O SharedPreferences mantém os dados em memória após o primeiro carregamento, tornando leituras subsequentes extremamente rápidas.
* Operações de escrita são assíncronas e não bloqueiam a UI.

**Limitações de Tamanho:**
* Não há limite oficial, mas recomenda-se armazenar apenas alguns kilobytes.
* Para dados maiores, considere alternativas como SQLite ou Hive.

**Segurança:**
* Os dados NÃO são criptografados.
* Não armazene informações sensíveis como senhas ou tokens de autenticação.
* Para dados sensíveis, use o pacote `flutter_secure_storage`.

**Tipos de Dados:**
* Apenas tipos primitivos são suportados.
* Para objetos complexos, você precisará serializá-los (geralmente em JSON) antes de salvá-los como string.

### Testando o Aplicativo

1. **Execute o aplicativo:**
   ```bash
   flutter run
   ```

2. Incremente o contador algumas vezes.

3. Feche completamente o aplicativo (não apenas minimize).

4. Reabra o aplicativo.

5. O contador deve exibir o último valor salvo, não zero.

### Boas Práticas

**1. Use constantes para chaves:**
```dart
class PrefsKeys {
  static const String counter = 'counter';
  static const String userName = 'user_name';
  static const String isDarkMode = 'is_dark_mode';
}
```
Criar uma classe dedicada para armazenar todas as chaves do seu aplicativo centraliza a configuração e facilita muito a manutenção. Assim, se você precisar mudar o nome de uma chave, basta alterar em um único lugar. Além disso, você tem uma visão clara de todos os dados que estão sendo armazenados no SharedPreferences. Para usar essas constantes, basta acessá-las através da classe: `prefs.setInt(PrefsKeys.counter, 5)` ou `prefs.getString(PrefsKeys.userName) ?? ''`.

**2. Crie uma classe utilitária:**
```dart
class PrefsService {
  static SharedPreferences? _prefs;
  
  static Future<void> init() async {
    _prefs ??= await SharedPreferences.getInstance();
  }
  
  static int getCounter() => _prefs?.getInt('counter') ?? 0;
  static Future<bool> setCounter(int value) async => 
      await _prefs!.setInt('counter', value);
}
```
Uma classe de serviço encapsula toda a lógica de acesso ao SharedPreferences em um único lugar. Isso torna o código mais organizado, pois você não precisa ficar obtendo a instância do SharedPreferences em vários lugares diferentes. Basta chamar `PrefsService.getCounter()` ou `PrefsService.setCounter(valor)` de qualquer lugar do seu código.

**3. Trate erros adequadamente:**
```dart
Future<void> _saveCounter() async {
  try {
    _prefs ??= await SharedPreferences.getInstance();
    final success = await _prefs!.setInt(_counterKey, _counter);
    if (!success) {
      // Lide com a falha
      print('Erro ao salvar o contador');
    }
  } catch (e) {
    print('Exceção ao salvar: $e');
  }
}
```
Operações de disco podem falhar por diversos motivos (espaço insuficiente, permissões, etc.). Usar `try-catch` permite que seu aplicativo lide com essas situações de forma elegante, mostrando uma mensagem ao usuário ou tentando novamente, ao invés de simplesmente travar.

**4. Inicialize no main() quando apropriado:**
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await PrefsService.init();
  runApp(const MyApp());
}
```
Se você criou uma classe utilitária como no exemplo 2, inicializar o SharedPreferences no `main()` garante que ele esteja pronto antes de qualquer tela ser carregada. Isso evita delays na primeira vez que você precisar acessar os dados. O `WidgetsFlutterBinding.ensureInitialized()` é necessário para usar plugins antes de chamar `runApp()`.

### Alternativas ao SharedPreferences

Dependendo das necessidades do seu projeto, considere:

* **flutter_secure_storage:** Para dados sensíveis que exigem criptografia
* **Hive:** Banco de dados NoSQL rápido e eficiente, ótimo para objetos complexos
* **sqflite:** Banco de dados SQL completo para consultas complexas
* **get_storage:** Alternativa mais rápida ao SharedPreferences
* **ObjectBox:** Banco de dados de alto desempenho para modelos de objetos

### Conclusão

O SharedPreferences é uma ferramenta fundamental no desenvolvimento Flutter, oferecendo uma forma simples e eficiente de persistir dados básicos. Embora tenha limitações, é perfeito para casos de uso como preferências do usuário, configurações da aplicação e estados simples.

Implementamos persistência no contador padrão do Flutter, demonstrando os conceitos fundamentais: inicialização assíncrona, carregamento de dados existentes e salvamento após mudanças de estado. Essa base pode ser expandida para cenários mais complexos em suas aplicações.

Lembre-se sempre de escolher a ferramenta certa para cada caso de uso: SharedPreferences para dados simples, soluções de armazenamento seguro para informações sensíveis, e bancos de dados completos para dados estruturados complexos.