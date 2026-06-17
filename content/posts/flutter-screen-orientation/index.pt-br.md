---
title: "Como Travar a Orientação de Tela no Flutter"
date: 2026-06-17T13:30:00-03:00
draft: false
tags: ["Flutter", "Dart", "SystemChrome", "Orientação"]
categories: ["Flutter"]
---

Muita gente que quer travar a orientação de um app Flutter vai direto no `AndroidManifest.xml` e adiciona `android:screenOrientation="portrait"`. Funciona, mas existe uma forma melhor.

### A abordagem Dart: `SystemChrome`

Adicione o import e configure as orientações logo no início da função `main()`, antes de chamar `runApp`:

```dart
import 'package:flutter/services.dart'; // ← import necessário

void main() {
  WidgetsFlutterBinding.ensureInitialized(); // precisa disso antes de mexer no sistema
  SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown,
  ]);
  runApp(const MyApp());
}
```

O `WidgetsFlutterBinding.ensureInitialized()` é necessário para garantir que o Flutter esteja pronto antes de qualquer chamada de sistema. Sem ele, o `setPreferredOrientations` pode não funcionar corretamente.

### Variações

**Somente retrato (sem virar de cabeça pra baixo):**
```dart
SystemChrome.setPreferredOrientations([
  DeviceOrientation.portraitUp,
]);
```

**Somente paisagem:**
```dart
SystemChrome.setPreferredOrientations([
  DeviceOrientation.landscapeLeft,
  DeviceOrientation.landscapeRight,
]);
```

**Liberar todas as orientações:**
```dart
SystemChrome.setPreferredOrientations(DeviceOrientation.values);
```

### Por que não usar o AndroidManifest?

| | AndroidManifest | SystemChrome (Dart) |
|---|---|---|
| Quando age | Compile-time | Runtime |
| Plataformas | Só Android | Android **e** iOS |
| Flexibilidade | App inteiro, fixo | Pode mudar por tela |

A vantagem real do `SystemChrome` aparece quando você precisa de comportamento diferente por tela, por exemplo: travar em retrato no app inteiro, mas liberar paisagem num player de vídeo ou minigame específico. Basta chamar `setPreferredOrientations` novamente na tela que precisar.

Para travar o app todo no retrato, os dois funcionam. Mas o Dart resolve liso e ainda vale pro iOS de graça.
