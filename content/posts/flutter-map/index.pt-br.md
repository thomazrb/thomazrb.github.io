---
title: "Criando um Mapa Interativo com Pins no Flutter usando flutter_map"
date: 2025-08-19T22:20:15-03:00
draft: false
tags: ["Flutter", "Mapas", "OpenStreetMap"]
categories: ["Flutter"]
---

Adicionar mapas a um aplicativo Flutter é uma funcionalidade poderosa, mas muitos desenvolvedores pensam que a única opção é o Google Maps. Felizmente, existem alternativas robustas e de código aberto, como o pacote **`flutter_map`**, que utiliza os dados do **OpenStreetMap** e oferece uma flexibilidade incrível.

Neste guia, vamos construir um aplicativo simples que exibe um mapa com pins (marcadores) customizados e interativos. É a base perfeita para qualquer projeto que precise de geolocalização, desde apps de entrega até guias turísticos.

### Por que usar o `flutter_map`?

* **É Open Source:** Totalmente gratuito e mantido pela comunidade.
* **Altamente Customizável:** Permite o uso de diversos provedores de mapas e a criação de marcadores com qualquer widget do Flutter.
* **Excelente Suporte Web:** Funciona perfeitamente em aplicações web, além de mobile e desktop.

### Passo 1: Criando o Projeto e Adicionando as Dependências

Vamos começar criando um novo projeto Flutter e adicionando os pacotes necessários.

1.  **Crie o projeto:**
    ```bash
    flutter create flutter_map_example
    ```

2.  **Navegue para a pasta do projeto:**
    ```bash
    cd flutter_map_example
    ```

3.  **Adicione as dependências:** Precisaremos de dois pacotes. O `flutter_map` para o mapa em si, e o `latlong2` para facilitar o manuseio de coordenadas geográficas.
    ```bash
    flutter pub add flutter_map latlong2
    ```
    Este comando adiciona as linhas necessárias ao seu arquivo `pubspec.yaml` e baixa os pacotes.

### Passo 2: A Estrutura do Mapa

A estrutura do `flutter_map` é baseada em camadas (`Layers`). Pense nisso como folhas de papel transparente empilhadas: a primeira é o mapa base, a segunda contém os pins, a terceira pode ter polígonos, e assim por diante.

Vamos substituir o conteúdo do arquivo `lib/main.dart` pelo nosso código. A estrutura básica será:

* **`FlutterMap`**: O widget principal que contém todo o mapa.
* **`MapOptions`**: Onde definimos o centro inicial do mapa e o nível de zoom.
* **`children`**: Uma lista de camadas que serão desenhadas no mapa.

Nossa primeira camada será a `TileLayer`, responsável por buscar e exibir as "peças" (tiles) do mapa do OpenStreetMap.

### Passo 3: Criando Pins Customizados e Interativos

A verdadeira magia do `flutter_map` está na `MarkerLayer`. Um `Marker` (pin) pode ter como `child` qualquer widget do Flutter. Isso nos dá liberdade total para criar marcadores que não são apenas um ícone, mas também contêm texto, botões e gestos.

No nosso exemplo, criaremos uma função auxiliar `_buildCityMarker` que retorna um `Marker`. Este marcador será composto por:

1.  Um **`InkWell`** para capturar o toque e fornecer feedback visual (como a mudança do cursor do mouse na web).
2.  Um `Column` para empilhar o ícone e o texto.
3.  Um `Icon` para o pino visual.
4.  Um `Container` com `Text` para exibir o nome da cidade.

### Passo 4: O Código Completo

Agora, vamos juntar tudo. Substitua todo o conteúdo do seu arquivo `lib/main.dart` pelo código abaixo. Ele está completo, comentado e pronto para rodar.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_map/flutter_map.dart';
import 'package:latlong2/latlong.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Mapa ES',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MapaCidadesEsPage(),
      debugShowCheckedModeBanner: false,
    );
  }
}

class MapaCidadesEsPage extends StatefulWidget {
  const MapaCidadesEsPage({super.key});

  @override
  State<MapaCidadesEsPage> createState() => _MapaCidadesEsPageState();
}

class _MapaCidadesEsPageState extends State<MapaCidadesEsPage> {
  // Coordenadas geográficas das cidades
  static const LatLng _pontoSaoMateus = LatLng(-18.7160, -39.8582);
  static const LatLng _pontoVitoria = LatLng(-20.3194, -40.3378);

  // Ponto central para inicializar o mapa entre as duas cidades
  static const LatLng _pontoCentral = LatLng(-19.5177, -40.0980);

  // Lista que vai conter nossos pins (marcadores)
  late final List<Marker> _markers;

  @override
  void initState() {
    super.initState();

    // Inicializa a lista de marcadores quando o widget é criado
    _markers = [
      // Pin de São Mateus
      _buildCityMarker(
        point: _pontoSaoMateus,
        cityName: 'São Mateus',
        color: Colors.red,
      ),

      // Pin de Vitória
      _buildCityMarker(
        point: _pontoVitoria,
        cityName: 'Vitória',
        color: Colors.blue,
      ),
    ];
  }

  /// Função auxiliar para criar um Marker (pin) customizado
  Marker _buildCityMarker({
    required LatLng point,
    required String cityName,
    required Color color,
  }) {
    return Marker(
      point: point,
      width: 95, // Largura fixa que acomoda bem o maior nome
      height: 65, // Altura fixa para o pin
      child: InkWell(
        onTap: () {
          // Ação ao clicar no pin: exibe uma notificação simples (SnackBar)
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('Você clicou em $cityName!'),
              backgroundColor: color,
              duration: const Duration(seconds: 2),
            ),
          );
        },
        // O conteúdo visual do pin é uma coluna com o ícone e o texto
        child: Column(
          children: [
            Icon(
              Icons.location_pin,
              color: color,
              size: 40,
            ),
            Container(
              padding: const EdgeInsets.symmetric(horizontal: 5, vertical: 2),
              decoration: BoxDecoration(
                color: Colors.white.withAlpha(204),
                borderRadius: BorderRadius.circular(10),
              ),
              child: Text(
                cityName,
                style: const TextStyle(
                  fontWeight: FontWeight.bold,
                  fontSize: 12,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Mapa - Espírito Santo'),
        centerTitle: true,
      ),
      // O widget principal que renderiza o mapa
      body: FlutterMap(
        // Opções do mapa, como centro e zoom inicial
        options: const MapOptions(
          initialCenter: _pontoCentral,
          initialZoom: 8.0, // Zoom que mostra as duas cidades
        ),
        // O mapa é construído em camadas (layers)
        children: [
          // Camada 1: O mapa base do OpenStreetMap
          TileLayer(
            urlTemplate: '[https://tile.openstreetmap.org/](https://tile.openstreetmap.org/){z}/{x}/{y}.png',
            userAgentPackageName: 'com.example.flutter_map_example',
          ),
          // Camada 2: Nossos pins (marcadores)
          MarkerLayer(
            markers: _markers,
          ),
        ],
      ),
    );
  }
}
```
### Uma Nota Importe sobre o OpenStreetMap

Ao rodar o aplicativo, você notará uma mensagem informativa no seu console de depuração. Ela não é um erro, mas sim um aviso importante da comunidade `flutter_map`: os servidores públicos do OpenStreetMap (OSM) não são feitos para uso comercial ou de alto tráfego.

O OSM é um projeto incrível mantido por voluntários, e seus servidores têm capacidade limitada. Para desenvolvimento, testes e projetos pessoais, seu uso é aceitável. No entanto, se você planeja lançar um aplicativo para um grande público, a prática recomendada é usar um provedor de "tiles" comercial, muitos dos quais oferecem generosos planos gratuitos. Mudar de provedor geralmente envolve apenas a troca da `urlTemplate` no seu `TileLayer`.

### Conclusão

E aí está! Com poucas linhas de código, criamos um mapa funcional e interativo no Flutter. A partir daqui, as possibilidades são enormes: você pode carregar os pins de uma API, navegar para uma tela de detalhes ao tocar em um marcador ou até mesmo usar outros pacotes do ecossistema `flutter_map` para adicionar pop-ups e clusters de marcadores.

O `flutter_map` prova ser uma ferramenta poderosa e acessível para qualquer desenvolvedor que queira integrar mapas em seus projetos sem depender de serviços proprietários.