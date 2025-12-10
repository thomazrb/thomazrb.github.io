---
title: "Riverpod: Gerenciamento de Estado Moderno e Simples no Flutter"
date: 2025-12-10T15:30:00-03:00
draft: false
tags: ["Flutter", "Riverpod", "State Management", "Dart"]
categories: ["Flutter"]
---

Quando desenvolvemos aplicativos Flutter, frequentemente precisamos lidar com informações que mudam ao longo do tempo: o número exibido em um contador, se o usuário está logado ou não, os itens em um carrinho de compras, o tema atual do app. Essas informações que podem mudar e precisam ser compartilhadas entre diferentes partes do aplicativo são o que chamamos de **estado**. Gerenciar estado significa controlar essas informações de forma organizada: onde elas ficam armazenadas, como são modificadas e como os widgets são notificados quando elas mudam para atualizar a interface.

Gerenciar o estado de um aplicativo Flutter pode parecer complicado no início, mas com o **Riverpod**, essa tarefa se torna muito mais simples e poderosa. Riverpod é uma evolução do Provider, criado pelo mesmo desenvolvedor (Remi Rousselet), trazendo melhorias significativas como não depender do `BuildContext` e melhor testabilidade.

Neste guia, vamos construir um aplicativo simples de contador para demonstrar como o Riverpod funciona na prática. Você aprenderá a criar providers, consumir seus valores e reagir a mudanças de estado de forma limpa e eficiente.

**Versões utilizadas neste tutorial:**
- Flutter: 3.38.3
- flutter_riverpod: 3.0.3

### Por que usar Riverpod?

* **Sem Context:** Você pode acessar providers de qualquer lugar, sem precisar do `BuildContext` como era necessário no Provider.
* **Segurança em Tempo de Compilação:** Erros de tipo são detectados durante a compilação, não em tempo de execução.
* **Testável:** Muito mais fácil de testar do que outras soluções de gerenciamento de estado.
* **Menos BoilerPlate:** Código mais limpo e direto que outras soluções como BLoC (Business Logic Component), outro padrão de gerenciamento de estado muito popular no Flutter. É ótimo para apps grandes e complexos, mas pode ser "demais" para projetos simples.
* **Evolução do Provider:** Criado pelo mesmo desenvolvedor, corrigindo problemas de design do Provider original.

### Passo 1: Criando o Projeto e Adicionando Dependências

Vamos começar criando um novo projeto Flutter e adicionando o Riverpod.

1. **Crie o projeto:**
   ```bash
   flutter create riverpod_example
   ```

2. **Navegue para a pasta do projeto:**
   ```bash
   cd riverpod_example
   ```

3. **Adicione o Riverpod:** Usaremos o `flutter_riverpod`, que é a versão específica para Flutter.
   ```bash
   flutter pub add flutter_riverpod
   ```

### Passo 2: Entendendo os Conceitos Básicos

Antes de mergulhar no código, vamos entender os principais conceitos do Riverpod de forma bem clara:

* **Notifier:** É uma classe especial que funciona como uma "estação de rádio". Quando o valor muda, ela transmite atualizações para todos os widgets que estão "sintonizados" nela. Pense nela como uma estação de rádio que transmite atualizações para todos os aparelhos sintonizados.

* **NotifierProvider:** É o widget que disponibiliza o `Notifier` para toda a aplicação. É como instalar a antena da estação de rádio - sem ela, ninguém consegue sintonizar o sinal. Geralmente declaramos ele em um arquivo separado de providers.

* **ProviderScope:** Um widget que deve envolver toda a sua aplicação no `main.dart`. Ele é o "gerente geral" que cuida de todos os providers. Sem ele, nada funciona - é como a torre de transmissão principal que coordena todas as antenas.

* **ConsumerWidget:** Um widget especial que "escuta" as transmissões do provider. Toda vez que o `Notifier` envia uma atualização, o `ConsumerWidget` reconstrói automaticamente. É como um rádio que automaticamente atualiza a informação na tela quando recebe um novo sinal. **Importante:** É dentro do `ConsumerWidget` que temos acesso ao `ref`, que nos permite usar `ref.watch` e `ref.read`. Se você usar um `StatelessWidget` normal, não terá acesso ao `ref`.

* **ref.watch():** Um método que sintoniza na "frequência" do provider e fica ouvindo as atualizações. Quando algo muda, o widget é reconstruído automaticamente. Use dentro do método `build()` quando você quer que a tela atualize sozinha. **Só funciona dentro de um `ConsumerWidget`!**

* **ref.read():** Este método apenas "dá uma olhadinha" no valor atual sem ficar ouvindo a transmissão. É como verificar rapidamente qual música está tocando sem ligar o rádio. Use dentro de callbacks (como `onPressed` de um botão ou `onChanged` de um TextField) quando você só quer executar uma ação pontual. **Também só funciona dentro de um `ConsumerWidget`!**

### Passo 3: Estrutura do Projeto

Vamos criar uma estrutura simples e organizada. Nosso app terá duas páginas para demonstrar como o Riverpod compartilha estado entre telas diferentes:

```
lib/
  ├── main.dart
  ├── providers/
  │   └── counter_provider.dart
  └── pages/
      ├── counter_page.dart      # Página com os botões
      └── display_page.dart      # Página que só exibe o valor
```

### Passo 4: Criando o Provider do Contador

#### `lib/providers/counter_provider.dart`

Vamos criar um `Notifier` bem simples. A partir do Riverpod 3.0+, não podemos mais acessar o `state` diretamente de fora do Notifier, então criamos um método `setValue`:

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Classe que gerencia o estado do contador
class CounterNotifier extends Notifier<int> {
  // Retorna o valor inicial
  @override
  int build() => 0;
  
  // Método para modificar o valor
  void setValue(int value) {
    state = value;
  }
}

// Provider que disponibiliza o CounterNotifier
final counterProvider = NotifierProvider<CounterNotifier, int>(() {
  return CounterNotifier();
});
```

**Como funciona:**
- **Para ler o valor:** `ref.watch(counterProvider)` - lê e fica observando mudanças (reconstrói o widget quando muda)
- **Para modificar o valor:** `ref.read(counterProvider.notifier).setValue(novoValor)` - acessa o notifier e chama o método para mudar o estado

**Por que `.notifier`?** Quando fazemos `ref.read(counterProvider)`, pegamos apenas o **valor** (o int, o número). Mas o valor sozinho não tem métodos. Para modificar, precisamos acessar o **gerenciador** (o `CounterNotifier`), que é quem tem os métodos como `setValue()`. Por isso usamos `ref.read(counterProvider.notifier)` - estamos pegando o gerenciador, não o valor.

O Riverpod cuida de avisar todos os widgets que estão usando `ref.watch` automaticamente quando você chama `setValue`.

### Passo 5: Criando as Páginas

Agora vamos criar duas páginas para demonstrar o poder do Riverpod: uma página com um campo de texto para digitar o valor do contador, e outra que apenas exibe o valor. O interessante é que **não vamos passar o valor como parâmetro** entre as páginas - o Riverpod cuida disso!

#### `lib/pages/counter_page.dart`

Esta é a página principal com o campo de texto para modificar o contador:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/counter_provider.dart';
import 'display_page.dart';

class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ref.watch observa o provider e reconstrói quando muda
    final counter = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Digite o Valor'),
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(32.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Text(
                'Valor atual:',
                style: TextStyle(fontSize: 20),
              ),
              const SizedBox(height: 16),
              Text(
                '$counter',
                style: const TextStyle(
                  fontSize: 72,
                  fontWeight: FontWeight.bold,
                  color: Colors.blue,
                ),
              ),
              const SizedBox(height: 32),
              TextField(
                decoration: const InputDecoration(
                  labelText: 'Digite um número',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.edit),
                ),
                keyboardType: TextInputType.number,
                onChanged: (value) {
                  // Tenta converter o texto para número
                  final numero = int.tryParse(value);
                  if (numero != null) {
                    // ref.read não reconstrói, apenas modifica o estado
                    ref.read(counterProvider.notifier).setValue(numero);
                  }
                },
              ),
              const SizedBox(height: 48),
              ElevatedButton.icon(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(builder: (context) => const DisplayPage()),
                  );
                },
                icon: const Icon(Icons.visibility),
                label: const Text('Ver em Outra Página'),
                style: ElevatedButton.styleFrom(
                  padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

#### `lib/pages/display_page.dart`

Esta página apenas exibe o valor do contador. Note que **não recebemos nenhum parâmetro** - o Riverpod fornece o valor automaticamente. Aqui vamos usar `ref.read` ao invés de `ref.watch` para demonstrar a diferença:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/counter_provider.dart';

class DisplayPage extends ConsumerWidget {
  const DisplayPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Aqui usamos ref.read - pega o valor UMA VEZ quando a página é criada
    final counter = ref.read(counterProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Visualização'),
        backgroundColor: Colors.green,
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(
              Icons.remove_red_eye,
              size: 64,
              color: Colors.green,
            ),
            const SizedBox(height: 24),
            const Text(
              'O contador está em:',
              style: TextStyle(fontSize: 24),
            ),
            const SizedBox(height: 16),
            Text(
              '$counter',
              style: const TextStyle(
                fontSize: 96,
                fontWeight: FontWeight.bold,
                color: Colors.green,
              ),
            ),
            const SizedBox(height: 48),
            Card(
              margin: const EdgeInsets.symmetric(horizontal: 32),
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  children: const [
                    Icon(Icons.info_outline, size: 48, color: Colors.green),
                    SizedBox(height: 16),
                    Text(
                      'Esta página usa ref.read',
                      style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                      textAlign: TextAlign.center,
                    ),
                    SizedBox(height: 8),
                    Text(
                      'Ela pega o valor atual quando é criada. Volte, mude o contador, e volte aqui - verá o novo valor!',
                      textAlign: TextAlign.center,
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**ref.watch vs ref.read na prática:**

- **CounterPage (primeira)** usa `ref.watch`: O número atualiza automaticamente enquanto você digita no TextField porque está "observando" as mudanças.

- **DisplayPage (segunda)** usa `ref.read`: Pega o valor apenas quando a página é criada. Como o Flutter **recria** a página toda vez que você navega pra ela, sempre verá o valor mais atual. Porém, se algo mudasse o contador enquanto você está olhando essa página, ela **não atualizaria** sozinha.

**Quando usar cada um:**
- Use `ref.watch` quando precisar que a tela atualize automaticamente ao detectar mudanças
- Use `ref.read` quando quiser pegar o valor uma vez só, sem ficar observando (economiza recursos)

### Passo 6: Configurando o Main

#### `lib/main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'pages/counter_page.dart';

void main() {
  // ProviderScope deve envolver todo o app
  // Ele é o "gerente geral" de todos os providers
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod Example',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
      ),
      home: const CounterPage(),
    );
  }
}
```

### Passo 7: Entendendo o Fluxo de Dados

1. **Criação do Provider:** Criamos uma classe `CounterNotifier` que estende `Notifier<int>` e define o valor inicial no método `build()`. Também criamos um método `setValue()` para modificar o estado.

2. **ProviderScope:** Envolvemos o app com `ProviderScope` no `main.dart` para que todos os providers estejam disponíveis em toda a aplicação.

3. **Compartilhamento entre Páginas:** Ambas as páginas (`CounterPage` e `DisplayPage`) acessam o **mesmo provider** através do `ref`. Não precisamos passar parâmetros no `Navigator.push()` - o Riverpod gerencia isso!

4. **Leitura com `ref.watch` e `ref.read`:** 
   - A `CounterPage` usa `ref.watch(counterProvider)` para observar mudanças - por isso o número atualiza automaticamente enquanto você digita no TextField
   - A `DisplayPage` usa `ref.read(counterProvider)` para pegar o valor apenas quando a página é criada - como o Flutter recria a página toda vez que você navega pra ela, sempre mostra o valor atual

5. **Modificação com `ref.read`:** Para modificar o estado, usamos `ref.read(counterProvider.notifier).setValue(novoValor)`. Simples e direto!

### `ref.watch` vs `ref.read`

É importante entender as diferentes formas de acessar o provider:

* **`ref.watch(counterProvider)`**: Lê o valor E se inscreve para mudanças. O widget será reconstruído automaticamente quando o estado mudar. Use no método `build()` (o método que constrói/desenha seu widget na tela) quando precisar que a tela atualize sozinha ao detectar mudanças. Quando você usa `ref.watch` dentro do `build()` de um widget, o Riverpod "registra" que aquele widget quer ser avisado de mudanças - então toda vez que o estado muda, o Riverpod chama o método `build()` de novo, redesenhando o widget com o novo valor.

* **`ref.read(counterProvider)`**: Lê o valor uma única vez, SEM se inscrever para mudanças. O widget não será reconstruído se o estado mudar. Use quando quiser apenas pegar o valor atual sem ficar observando (economiza recursos). No nosso exemplo, usamos isso na `DisplayPage`.

* **`ref.read(counterProvider.notifier).setValue()`**: Acessa o notifier (o gerenciador) para chamar métodos que modificam o estado. Use dentro de callbacks (como `onPressed` de um botão ou `onChanged` de um TextField) quando você quer apenas executar uma ação de modificação, sem reconstruir o widget. Nunca use `ref.watch` dentro de callbacks - isso causaria problemas ao tentar reconstruir o widget no meio de uma ação.

### Exemplo Prático

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {  // <-- Método build do widget
  // watch: reconstrói quando counter muda (observa mudanças)
  // Quando o estado muda, este método build() roda de novo
  final counter = ref.watch(counterProvider);
  
  // read: pega o valor uma vez, não observa mudanças
  // final counter = ref.read(counterProvider);
  
  return Column(
    children: [
      Text('$counter'), // Atualiza automaticamente se usou watch
      
      TextField(
        onChanged: (value) {  // <-- Callback, não é o build()
          final numero = int.tryParse(value);
          if (numero != null) {
            // read + notifier: acessa o gerenciador para modificar
            // Aqui usamos read porque estamos dentro de um callback
            ref.read(counterProvider.notifier).setValue(numero);
          }
        },
      ),
    ],
  );
}
```

### Trabalhando com Múltiplos Valores

Uma dúvida comum: e se eu precisar armazenar mais de um valor no meu provider? Por exemplo, três inteiros ao invés de um? Com Riverpod, você pode criar uma classe para agrupar valores relacionados:

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Classe para agrupar valores relacionados
class CountersState {
  final int counter1;
  final int counter2;
  final int counter3;

  CountersState({
    required this.counter1,
    required this.counter2,
    required this.counter3,
  });
}

// Notifier que gerencia o estado composto
class CountersNotifier extends Notifier<CountersState> {
  @override
  CountersState build() {
    return CountersState(counter1: 0, counter2: 0, counter3: 0);
  }
  
  // Métodos para modificar valores individuais
  void setCounter1(int value) {
    state = CountersState(
      counter1: value,
      counter2: state.counter2,
      counter3: state.counter3,
    );
  }
  
  void setCounter2(int value) {
    state = CountersState(
      counter1: state.counter1,
      counter2: value,
      counter3: state.counter3,
    );
  }
  
  void setCounter3(int value) {
    state = CountersState(
      counter1: state.counter1,
      counter2: state.counter2,
      counter3: value,
    );
  }
}

// Provider
final countersProvider = NotifierProvider<CountersNotifier, CountersState>(() {
  return CountersNotifier();
});
```

**Como usar:**

```dart
// Ler valores
final state = ref.watch(countersProvider);
Text('Counter 1: ${state.counter1}');

// Modificar valores
ref.read(countersProvider.notifier).setCounter1(10);
```

**Dica:** Para facilitar a modificação de apenas um valor, você pode adicionar um método `copyWith` na classe `CountersState`, como mostramos na seção de boas práticas a seguir.

### Boas Práticas com Riverpod

1. **Mantenha os Providers Organizados:** Crie uma pasta `providers` e separe por funcionalidade.

2. **Evite Lógica no Build:** Se precisar de lógica complexa, crie métodos no `Notifier` ao invés de manipular o estado diretamente.

3. **Use `ConsumerWidget` apenas quando necessário:** Se um widget não precisa acessar providers, pode ser um `StatelessWidget` normal.

4. **Use `autoDispose` quando apropriado:** Por padrão, os providers ficam "vivos" na memória o tempo todo, mesmo que nenhum widget esteja usando. O `autoDispose` faz com que o provider seja automaticamente destruído quando o último widget que estava usando ele for removido da tela. Isso economiza memória!
   ```dart
   // SEM autoDispose: fica na memória para sempre
   final counterProvider = NotifierProvider<CounterNotifier, int>(() {
     return CounterNotifier();
   });
   
   // COM autoDispose: é limpo quando nenhum widget está mais usando
   final tempProvider = NotifierProvider.autoDispose<CounterNotifier, int>(() {
     return CounterNotifier();
   });
   ```
   **Quando usar autoDispose:**
   - ✅ Dados temporários (detalhes de produto, perfil de usuário visitado, etc)
   - ✅ Dados específicos de uma tela que não precisam ficar na memória
   - ❌ Dados globais que você quer manter (usuário logado, tema do app, carrinho de compras)

5. **Crie métodos auxiliares para estados complexos:** Se seu estado tem muitos campos, crie um método `copyWith`:
   ```dart
   class CountersState {
     final int counter1;
     final int counter2;
     
     CountersState({required this.counter1, required this.counter2});
     
     CountersState copyWith({int? counter1, int? counter2}) {
       return CountersState(
         counter1: counter1 ?? this.counter1,
         counter2: counter2 ?? this.counter2,
       );
     }
   }
   
   // Uso: facilita atualizar apenas um campo
   state = state.copyWith(counter1: 10);
   ```

### Conclusão

Riverpod transforma o gerenciamento de estado no Flutter em algo simples e poderoso. Com sua arquitetura sem `BuildContext`, segurança de tipos e facilidade de testes, ele se tornou a escolha preferida de muitos desenvolvedores Flutter modernos.

O exemplo que construímos demonstra um dos poderes mais importantes do gerenciamento de estado: **compartilhar estado entre páginas diferentes sem passar parâmetros**. As duas páginas acessam o mesmo provider através do `ref` e reagem automaticamente às mudanças. Isso é gerenciamento de estado na prática!

A partir daqui, você tem uma base sólida para construir aplicações Flutter escaláveis e bem estruturadas. Experimente adicionar mais funcionalidades ao app, como salvar o contador no SharedPreferences ou criar múltiplos contadores independentes. O código está pronto para evoluir!

**Recursos Adicionais:**
* [Documentação Oficial do Riverpod](https://riverpod.dev)
* [Riverpod no Pub.dev](https://pub.dev/packages/flutter_riverpod)
* [Exemplos Oficiais](https://github.com/rrousselGit/riverpod/tree/master/examples)