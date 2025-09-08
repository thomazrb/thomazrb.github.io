---
title: "Tutorial Detalhado: Criando um Widget de Calendário Interativo em Flutter"
date: 2025-09-07T10:52:27-03:00
draft: false
tags: ["Flutter", "Custom Widgets", "Dart", "UI", "State Management"]
categories: ["Flutter"]
---

Em muitos aplicativos, um calendário é mais do que apenas uma grade de datas; ele é uma ferramenta de agendamento, um planejador de eventos ou uma forma de visualizar dados ao longo do tempo. Embora existam pacotes prontos, construir seu próprio widget de calendário no Flutter oferece um controle incomparável sobre a aparência, a funcionalidade e a lógica de negócios.

Neste tutorial, vamos mergulhar fundo no processo de criação de um widget de calendário mensal. Vamos dissecar o código-fonte trecho por trecho, explicando a lógica por trás de cada parte. No final, apresentaremos os arquivos completos e limpos, prontos para serem copiados para o seu projeto.

O código-fonte completo deste projeto está disponível no GitHub para consulta:
**[Acesse o repositório completo aqui!](https://github.com/thomazrb/flutter_calendar_example)**

### A Arquitetura: Separando Responsabilidades

A chave para um bom componente reutilizável é a separação de responsabilidades:

1.  **`main.dart` (A Tela Principal):** É o "cérebro". Ele é responsável por gerenciar o estado, criar os dados e decidir o que fazer quando um dia é clicado.
2.  **`calendario_mensal.dart` (O Widget Reutilizável):** É a "mão de obra". Ele apenas recebe dados, os desenha na tela e avisa ao "cérebro" quando uma interação acontece.

Vamos analisar o código de cada um.

### Parte 1: O Cérebro da Aplicação (`main.dart`)

Este arquivo configura a tela principal que orquestra o uso do nosso calendário.

#### Trecho 1: Estrutura Básica e Gerenciamento de Estado

Primeiro, criamos a estrutura do app e definimos nossa `HomePage` como um `StatefulWidget`. Isso é crucial porque a tela precisa "lembrar" qual data foi selecionada pelo usuário, e essa informação é a nossa "variável de estado", `_dataSelecionada`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_calendar_example/widgets/calendario_mensal.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exemplo de Calendário',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  DateTime? _dataSelecionada;

  @override
  Widget build(BuildContext context) {
    // ... O resto da UI virá aqui
    return Scaffold(); // Placeholder
  }
}
```

#### Trecho 2: Preparando os Dados

Dentro do método `build`, preparamos os dados que serão passados para o calendário. Em um aplicativo real, eles viriam de uma API ou banco de dados. Aqui, criamos mapas de exemplo para os eventos e as cores. Usar `DateTime` como chave é a melhor prática, pois torna os dados inequívocos.

```dart
// Dentro do método build() de _HomePageState

final hoje = DateTime.now();
final anoAtual = hoje.year;
final mesAtual = hoje.month;

const nomesDosMeses = [
  'Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho',
  'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'
];

final Map<DateTime, Widget> eventosDoMes = {
  DateTime(anoAtual, mesAtual, 5): const Icon(Icons.star, color: Colors.amber, size: 24),
  DateTime(anoAtual, mesAtual, 13): const Text("TXT", style: TextStyle(fontSize: 12, fontWeight: FontWeight.bold, color: Colors.blue)),
  DateTime(anoAtual, mesAtual, 20): Column(
    mainAxisAlignment: MainAxisAlignment.center,
    children: const [
      Text("L1", style: TextStyle(fontSize: 9)),
      Text("L2", style: TextStyle(fontSize: 9)),
    ],
  ),
  DateTime(anoAtual, mesAtual, 22): const Icon(Icons.favorite, color: Colors.pink, size: 24),
};

final Map<DateTime, Color> coresDoMes = {
  DateTime(anoAtual, mesAtual, 1): Colors.blue[100]!,
  DateTime(anoAtual, mesAtual, 7): Colors.red[100]!,
  DateTime(anoAtual, mesAtual, 13): Colors.orange[100]!,
  DateTime(anoAtual, mesAtual, 14): Colors.red[100]!,
  DateTime(anoAtual, mesAtual, 21): Colors.red[100]!,
  DateTime(anoAtual, mesAtual, 22): Colors.green[100]!,
  DateTime(anoAtual, mesAtual, 28): Colors.red[200]!,
};
```

#### Trecho 3: Construindo a UI e Usando o Calendário

Agora, construímos a `Scaffold` e chamamos nosso widget `CalendarioMensal`, passando todos os dados que preparamos.

```dart
// Dentro do método build(), retornando a Scaffold

return Scaffold(
  appBar: AppBar(
    title: const Text('Meu Calendário Interativo'),
    backgroundColor: Colors.blueGrey[800],
    foregroundColor: Colors.white,
  ),
  body: Padding(
    padding: const EdgeInsets.all(16.0),
    child: Column(
      children: [
        Text(
          '${nomesDosMeses[mesAtual - 1]} de $anoAtual',
          style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 20),
        Center(
          child: CalendarioMensal(
            ano: anoAtual,
            mes: mesAtual,
            largura: MediaQuery.of(context).size.width / 2,
            conteudoDosDias: eventosDoMes,
            coresDosDias: coresDoMes,
            diaSelecionado: _dataSelecionada,
            onDiaSelecionado: (data) {
              // ... O callback virá no próximo trecho
            },
          ),
        ),
        // ... O texto de feedback virá aqui
      ],
    ),
  ),
);
```

#### Trecho 4: Implementando a Interatividade com o Callback (`onDiaSelecionado`)

Este é o coração da interatividade. Passamos uma função para o parâmetro `onDiaSelecionado`. Quando um dia é clicado dentro do calendário, essa função é executada aqui na tela principal. Dentro dela, usamos `setState` para atualizar nossa variável `_dataSelecionada`, o que força o Flutter a redesenhar a tela com a nova informação.

```dart
// Parâmetro onDiaSelecionado e o Text de feedback

// ... dentro do Center()
child: CalendarioMensal(
  // ... outros parâmetros
  onDiaSelecionado: (data) {
    setState(() {
      _dataSelecionada = data;
    });

    ScaffoldMessenger.of(context).hideCurrentSnackBar();
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Data selecionada: ${data.day}/${data.month}/${data.year}'),
        duration: const Duration(seconds: 1),
      ),
    );
  },
),
// ...
const SizedBox(height: 20),
Text(
  _dataSelecionada == null
      ? 'Nenhum dia selecionado'
      : 'Dia selecionado: ${_dataSelecionada!.day}/${_dataSelecionada!.month}/${_dataSelecionada!.year}',
  style: const TextStyle(fontSize: 18, fontWeight: FontWeight.w500),
),
```

---

### Parte 2: A Mão de Obra (`calendario_mensal.dart`)

Agora, vamos dissecar o widget reutilizável que faz todo o trabalho de desenho.

#### Trecho 1: Estrutura e Parâmetros

Definimos a classe `CalendarioMensal` e todos os seus parâmetros de entrada (propriedades). Este é o "contrato" do widget: as informações que ele precisa receber para funcionar.

```dart
import 'package:flutter/material.dart';

class CalendarioMensal extends StatelessWidget {
  final int ano;
  final int mes;
  final double largura;
  final Map<DateTime, Widget>? conteudoDosDias;
  final Map<DateTime, Color>? coresDosDias;
  final void Function(DateTime)? onDiaSelecionado;
  final DateTime? diaSelecionado;

  const CalendarioMensal({
    super.key,
    required this.ano,
    required this.mes,
    required this.largura,
    this.conteudoDosDias,
    this.coresDosDias,
    this.onDiaSelecionado,
    this.diaSelecionado,
  });
  
  @override
  Widget build(BuildContext context) {
    // ... Lógica principal
  }
}
```

#### Trecho 2: Lógica Principal no Método `build`

O método `build` começa com a lógica "inteligente": ele calcula quantos dias o mês tem e em que dia da semana ele começa. Em seguida, ele filtra os mapas de eventos e cores para pegar apenas os dados relevantes para o mês e ano atuais.

```dart
// Dentro do build() do CalendarioMensal

// 1. Cálculos de Data
final int quantosDiasTemNoMes = DateTime(ano, mes + 1, 0).day;
final primeiroDiaDoMes = DateTime(ano, mes, 1);
final int diaDaSemanaQueComeca = (primeiroDiaDoMes.weekday % 7) + 1;

// 2. Filtragem de Dados
final Map<int, Widget> conteudoFiltrado = {};
if (conteudoDosDias != null) {
  for (final evento in conteudoDosDias!.entries) {
    if (evento.key.year == ano && evento.key.month == mes) {
      conteudoFiltrado[evento.key.day] = evento.value;
    }
  }
}

final Map<int, Color> coresFiltradas = {};
if (coresDosDias != null) {
  for (final cor in coresDosDias!.entries) {
    if (cor.key.year == ano && cor.key.month == mes) {
      coresFiltradas[cor.key.day] = cor.value;
    }
  }
}
// ... O resto da construção da UI virá aqui
```

#### Trecho 3: Funções Auxiliares (`_gerarListaDeDias` e `_CabecalhoDiasDaSemana`)

Para manter o método `build` limpo, delegamos algumas tarefas a funções e widgets auxiliares.

**`_gerarListaDeDias`**: Esta função cria a lista de itens que a grade irá exibir. Ela calcula quantos espaços vazios (`null`) são necessários no início do mês e depois adiciona os dias de 1 até o final do mês.

```dart
// Função auxiliar dentro de CalendarioMensal
List<int?> _gerarListaDeDias(int diaDaSemanaQueComeca, int quantosDiasTemNoMes) {
  final List<int?> dias = [];
  final int diasVaziosNoInicio = diaDaSemanaQueComeca - 1;
  for (int i = 0; i < diasVaziosNoInicio; i++) {
    dias.add(null);
  }
  for (int i = 1; i <= quantosDiasTemNoMes; i++) {
    dias.add(i);
  }
  return dias;
}
```

**`_CabecalhoDiasDaSemana`**: Este é um widget simples e stateless que apenas desenha a linha com as iniciais dos dias da semana (D, S, T, Q, Q, S, S), garantindo que cada inicial ocupe o mesmo espaço de uma célula do calendário.

```dart
class _CabecalhoDiasDaSemana extends StatelessWidget {
  final double tamanhoDaCelula;
  const _CabecalhoDiasDaSemana({required this.tamanhoDaCelula});

  @override
  Widget build(BuildContext context) {
    const dias = ['D', 'S', 'T', 'Q', 'Q', 'S', 'S'];
    return Row(
      children: dias.map((dia) {
        return SizedBox(
          width: tamanhoDaCelula,
          child: Text(
            dia,
            textAlign: TextAlign.center,
            style: const TextStyle(
              fontWeight: FontWeight.bold,
              color: Colors.black54,
            ),
          ),
        );
      }).toList(),
    );
  }
}
```

#### Trecho 4: Construindo e Desenhando a Grade de Células (`_GridDoCalendario`)

Este widget é o coração visual do nosso calendário. Ele foi projetado para ser "burro", ou seja, não contém nenhuma lógica de data complexa; ele apenas recebe uma lista de dias e dados já processados e os desenha em uma grade. A ferramenta principal para isso é o `GridView.builder`.

```dart
class _GridDoCalendario extends StatelessWidget {
  // ... parâmetros ...

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 7,
      ),
      itemCount: dias.length,
      itemBuilder: (context, index) {
        // ... Lógica para construir cada célula ...
      },
    );
  }
}
```

**Dissecando o `GridView.builder`:**

* **`GridView.builder`**: Usamos este construtor porque ele é a forma mais eficiente de criar grades no Flutter. Ele constrói seus filhos (`itemBuilder`) sob demanda, à medida que eles se tornam visíveis na tela. Para nosso calendário, a lista é pequena, mas é uma excelente prática.
* **`shrinkWrap: true` e `physics: const NeverScrollableScrollPhysics()`**: Estas duas propriedades são essenciais porque estamos colocando um `GridView` (que é um widget rolável) dentro de um `Column`. `shrinkWrap` força a grade a ocupar apenas o espaço necessário, e `NeverScrollableScrollPhysics` desabilita a rolagem própria da grade, evitando conflitos de rolagem.
* **`gridDelegate`**: É aqui que definimos o layout da grade. `SliverGridDelegateWithFixedCrossAxisCount(crossAxisCount: 7)` instrui a grade a ter exatamente 7 colunas, o que é perfeito para uma semana de calendário.
* **`itemBuilder`**: Esta é a função que é chamada para cada item na nossa lista `dias`. Ela recebe o `index` do item e deve retornar o widget que o representa. É aqui que a lógica de cada célula acontece.

#### Trecho 5: A Lógica de Construção de Cada Célula (`itemBuilder`)

Agora, vamos analisar o código dentro do `itemBuilder`, que é executado para cada dia do mês.

```dart
// Trecho do itemBuilder dentro de _GridDoCalendario

final dia = dias[index];

if (dia == null) {
  return const SizedBox.shrink();
}

final Widget? conteudoDoDia = (conteudoDosDias != null && conteudoDosDias!.containsKey(dia)) ? conteudoDosDias![dia] : null;
final Color corDeFundo = (coresDosDias != null && coresDosDias!.containsKey(dia)) ? coresDosDias![dia]! : Colors.grey[200]!;

final bool isSelecionado = diaSelecionado != null &&
    diaSelecionado!.year == ano &&
    diaSelecionado!.month == mes &&
    diaSelecionado!.day == dia;

return Material(
  color: Colors.transparent,
  child: InkWell(
    borderRadius: BorderRadius.circular(4),
    onTap: () {
      if (onDiaSelecionado != null) {
        final dataClicada = DateTime(ano, mes, dia);
        onDiaSelecionado!(dataClicada);
      }
    },
    child: Container(
      height: tamanhoDaCelula,
      width: tamanhoDaCelula,
      margin: const EdgeInsets.all(2),
      decoration: BoxDecoration(
        color: corDeFundo,
        borderRadius: BorderRadius.circular(4),
        border: isSelecionado
            ? Border.all(color: Colors.blueGrey[700]!, width: 2.5)
            : Border.all(color: Colors.black12, width: 1),
      ),
      child: Stack(
        children: [
          Positioned(
            top: 4,
            right: 4,
            child: Text(
              dia.toString(),
              style: const TextStyle(
                fontSize: 12,
                fontWeight: FontWeight.w500,
                color: Colors.black54,
              ),
            ),
          ),
          if (conteudoDoDia != null)
            Center(child: conteudoDoDia),
        ],
      ),
    ),
  ),
);
```

**Análise do `itemBuilder`:**

1.  **`if (dia == null)`**: A primeira verificação lida com os espaços vazios no início do mês. Se o item da lista for `null`, retornamos um `SizedBox.shrink()`, que é um widget vazio e sem dimensões.
2.  **Busca de Dados**: Buscamos o `conteudoDoDia` e a `corDeFundo` nos mapas que o widget recebeu. Se não houver uma entrada para o dia atual, usamos um valor padrão (nulo para o conteúdo e cinza para o fundo).
3.  **Lógica de Seleção**: A variável `isSelecionado` se torna `true` apenas se a `dataSelecionada` recebida do widget pai corresponder exatamente ao ano, mês e dia da célula que está sendo construída.
4.  **Interatividade (`InkWell`)**: Cada célula é envolvida em um `InkWell` para capturar toques. No `onTap`, executamos o callback `onDiaSelecionado`, passando o `DateTime` completo do dia clicado. É assim que a célula "conversa" de volta com a tela principal.
5.  **Visual (`Container` e `Stack`)**: O `Container` é responsável pela aparência da célula: cor de fundo, bordas arredondadas e uma borda de destaque condicional se `isSelecionado` for verdadeiro. Finalmente, um `Stack` é usado para sobrepor o número do dia (posicionado no canto com `Positioned`) e o conteúdo customizado (centralizado com `Center`).

#### Detalhe Técnico: Por que usar `Material` antes do `InkWell`?

Uma pergunta comum ao ver o código acima é: por que o `InkWell` está envolvido por um widget `Material`?

A resposta está em como o Flutter desenha os efeitos visuais do Material Design. O `InkWell` é responsável por criar o efeito de ondulação (*ripple effect*) quando é tocado, mas ele não desenha esse efeito em si mesmo. Em vez disso, ele procura o ancestral `Material` mais próximo na árvore de widgets e "pede" para que ele desenhe a ondulação.

Se tivéssemos apenas o `InkWell` envolvendo nosso `Container` colorido, o efeito de ondulação ficaria invisível, pois seria desenhado *atrás* da cor de fundo do `Container`. Ao adicionar um `Material` transparente como pai, damos ao `InkWell` uma "tela de pintura" dedicada para desenhar o efeito por cima de tudo, garantindo que ele seja visível para o usuário.

---

### Código-Fonte Completo (Sem Comentários)

Aqui estão os dois arquivos completos, prontos para você copiar e colar no seu projeto. Para uma versão totalmente comentada de ambos os arquivos, explicando cada linha de código em detalhe, por favor, acesse o [repositório do projeto no GitHub](https://github.com/thomazrb/flutter_calendar_example).

#### `lib/main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_calendar_example/widgets/calendario_mensal.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Exemplo de Calendário',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  DateTime? _dataSelecionada;

  @override
  Widget build(BuildContext context) {
    final hoje = DateTime.now();
    final anoAtual = hoje.year;
    final mesAtual = hoje.month;

    const nomesDosMeses = [
      'Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho',
      'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'
    ];

    final Map<DateTime, Widget> eventosDoMes = {
      DateTime(anoAtual, mesAtual, 5): const Icon(Icons.star, color: Colors.amber, size: 24),
      DateTime(anoAtual, mesAtual, 13): const Text("TXT", style: TextStyle(fontSize: 12, fontWeight: FontWeight.bold, color: Colors.blue)),
      DateTime(anoAtual, mesAtual, 20): Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: const [
          Text("L1", style: TextStyle(fontSize: 9)),
          Text("L2", style: TextStyle(fontSize: 9)),
        ],
      ),
      DateTime(anoAtual, mesAtual, 22): const Icon(Icons.favorite, color: Colors.pink, size: 24),
    };

    final Map<DateTime, Color> coresDoMes = {
      DateTime(anoAtual, mesAtual, 1): Colors.blue[100]!,
      DateTime(anoAtual, mesAtual, 7): Colors.red[100]!,
      DateTime(anoAtual, mesAtual, 13): Colors.orange[100]!,
      DateTime(anoAtual, mesAtual, 14): Colors.red[100]!,
      DateTime(anoAtual, mesAtual, 21): Colors.red[100]!,
      DateTime(anoAtual, mesAtual, 22): Colors.green[100]!,
      DateTime(anoAtual, mesAtual, 28): Colors.red[200]!,
    };

    return Scaffold(
      appBar: AppBar(
        title: const Text('Meu Calendário Interativo'),
        backgroundColor: Colors.blueGrey[800],
        foregroundColor: Colors.white,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            Text(
              '${nomesDosMeses[mesAtual - 1]} de $anoAtual',
              style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 20),
            Center(
              child: CalendarioMensal(
                ano: anoAtual,
                mes: mesAtual,
                largura: MediaQuery.of(context).size.width / 2,
                conteudoDosDias: eventosDoMes,
                coresDosDias: coresDoMes,
                diaSelecionado: _dataSelecionada,
                onDiaSelecionado: (data) {
                  setState(() {
                    _dataSelecionada = data;
                  });
                  ScaffoldMessenger.of(context).hideCurrentSnackBar();
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(
                      content: Text('Data selecionada: ${data.day}/${data.month}/${data.year}'),
                      duration: const Duration(seconds: 1),
                    ),
                  );
                },
              ),
            ),
            const SizedBox(height: 20),
            Text(
              _dataSelecionada == null
                  ? 'Nenhum dia selecionado'
                  : 'Dia selecionado: ${_dataSelecionada!.day}/${_dataSelecionada!.month}/${_dataSelecionada!.year}',
              style: const TextStyle(fontSize: 18, fontWeight: FontWeight.w500),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### `lib/widgets/calendario_mensal.dart`

```dart
import 'package:flutter/material.dart';

class CalendarioMensal extends StatelessWidget {
  final int ano;
  final int mes;
  final double largura;
  final Map<DateTime, Widget>? conteudoDosDias;
  final Map<DateTime, Color>? coresDosDias;
  final void Function(DateTime)? onDiaSelecionado;
  final DateTime? diaSelecionado;

  const CalendarioMensal({
    super.key,
    required this.ano,
    required this.mes,
    required this.largura,
    this.conteudoDosDias,
    this.coresDosDias,
    this.onDiaSelecionado,
    this.diaSelecionado,
  }) : assert(mes >= 1 && mes <= 12, 'O mês deve ser entre 1 e 12.');

  @override
  Widget build(BuildContext context) {
    final int quantosDiasTemNoMes = DateTime(ano, mes + 1, 0).day;
    final primeiroDiaDoMes = DateTime(ano, mes, 1);
    final int diaDaSemanaQueComeca = (primeiroDiaDoMes.weekday % 7) + 1;

    final Map<int, Widget> conteudoFiltrado = {};
    if (conteudoDosDias != null) {
      for (final evento in conteudoDosDias!.entries) {
        if (evento.key.year == ano && evento.key.month == mes) {
          conteudoFiltrado[evento.key.day] = evento.value;
        }
      }
    }

    final Map<int, Color> coresFiltradas = {};
    if (coresDosDias != null) {
      for (final cor in coresDosDias!.entries) {
        if (cor.key.year == ano && cor.key.month == mes) {
          coresFiltradas[cor.key.day] = cor.value;
        }
      }
    }

    final List<int?> diasParaExibir = _gerarListaDeDias(
      diaDaSemanaQueComeca,
      quantosDiasTemNoMes,
    );
    final double tamanhoDaCelula = largura / 7;

    return SizedBox(
      width: largura,
      child: Column(
        children: [
          _CabecalhoDiasDaSemana(tamanhoDaCelula: tamanhoDaCelula),
          const SizedBox(height: 4),
          _GridDoCalendario(
            ano: ano,
            mes: mes,
            dias: diasParaExibir,
            tamanhoDaCelula: tamanhoDaCelula,
            conteudoDosDias: conteudoFiltrado,
            coresDosDias: coresFiltradas,
            onDiaSelecionado: onDiaSelecionado,
            diaSelecionado: diaSelecionado,
          ),
        ],
      ),
    );
  }

  List<int?> _gerarListaDeDias(int diaDaSemanaQueComeca, int quantosDiasTemNoMes) {
    final List<int?> dias = [];
    final int diasVaziosNoInicio = diaDaSemanaQueComeca - 1;
    for (int i = 0; i < diasVaziosNoInicio; i++) {
      dias.add(null);
    }
    for (int i = 1; i <= quantosDiasTemNoMes; i++) {
      dias.add(i);
    }
    return dias;
  }
}

class _CabecalhoDiasDaSemana extends StatelessWidget {
  final double tamanhoDaCelula;
  const _CabecalhoDiasDaSemana({required this.tamanhoDaCelula});

  @override
  Widget build(BuildContext context) {
    const dias = ['D', 'S', 'T', 'Q', 'Q', 'S', 'S'];
    return Row(
      children: dias.map((dia) {
        return SizedBox(
          width: tamanhoDaCelula,
          child: Text(
            dia,
            textAlign: TextAlign.center,
            style: const TextStyle(
              fontWeight: FontWeight.bold,
              color: Colors.black54,
            ),
          ),
        );
      }).toList(),
    );
  }
}

class _GridDoCalendario extends StatelessWidget {
  final int ano;
  final int mes;
  final List<int?> dias;
  final double tamanhoDaCelula;
  final Map<int, Widget>? conteudoDosDias;
  final Map<int, Color>? coresDosDias;
  final void Function(DateTime)? onDiaSelecionado;
  final DateTime? diaSelecionado;

  const _GridDoCalendario({
    required this.ano,
    required this.mes,
    required this.dias,
    required this.tamanhoDaCelula,
    this.conteudoDosDias,
    this.coresDosDias,
    this.onDiaSelecionado,
    this.diaSelecionado,
  });

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 7,
      ),
      itemCount: dias.length,
      itemBuilder: (context, index) {
        final dia = dias[index];

        if (dia == null) {
          return const SizedBox.shrink();
        }

        final Widget? conteudoDoDia = (conteudoDosDias != null && conteudoDosDias!.containsKey(dia)) ? conteudoDosDias![dia] : null;
        final Color corDeFundo = (coresDosDias != null && coresDosDias!.containsKey(dia)) ? coresDosDias![dia]! : Colors.grey[200]!;

        final bool isSelecionado = diaSelecionado != null &&
            diaSelecionado!.year == ano &&
            diaSelecionado!.month == mes &&
            diaSelecionado!.day == dia;

        return Material(
          color: Colors.transparent,
          child: InkWell(
            borderRadius: BorderRadius.circular(4),
            onTap: () {
              if (onDiaSelecionado != null) {
                final dataClicada = DateTime(ano, mes, dia);
                onDiaSelecionado!(dataClicada);
              }
            },
            child: Container(
              height: tamanhoDaCelula,
              width: tamanhoDaCelula,
              margin: const EdgeInsets.all(2),
              decoration: BoxDecoration(
                color: corDeFundo,
                borderRadius: BorderRadius.circular(4),
                border: isSelecionado
                    ? Border.all(color: Colors.blueGrey[700]!, width: 2.5)
                    : Border.all(color: Colors.black12, width: 1),
              ),
              child: Stack(
                children: [
                  Positioned(
                    top: 4,
                    right: 4,
                    child: Text(
                      dia.toString(),
                      style: const TextStyle(
                        fontSize: 12,
                        fontWeight: FontWeight.w500,
                        color: Colors.black54,
                      ),
                    ),
                  ),
                  if (conteudoDoDia != null)
                    Center(
                      child: conteudoDoDia,
                    ),
                ],
              ),
            ),
          ),
        );
      },
    );
  }
}
```
### Conclusão

Construímos um sistema de componentes coeso e desacoplado. A tela principal gerencia o estado e os dados, enquanto o widget de calendário se concentra exclusivamente em exibir esses dados e relatar interações do usuário.

Essa arquitetura de passar dados para baixo e emitir eventos para cima (através de callbacks) é um dos padrões mais fundamentais e poderosos no desenvolvimento com Flutter, permitindo a criação de UIs complexas e, ao mesmo tempo, organizadas e fáceis de manter.