---
title: "Como Remover o Banner de Debug do Flutter em um Único Passo"
date: 2023-05-07T21:53:18-03:00
lastmod: 2023-05-28T20:24:21-03:00
tags: ["Flutter", "Debug Banner"]
categories: ["Flutter"]
---

## Atualizações:

*  *Código atualizado para Flutter 3.10.*

---

![Flutter Debug Banner](debug-banner.png)

Toda aplicação Flutter vem por padrão no modo de depuração, o que significa que a faixa de debug é exibida. Ela serve apenas para nos lembrar de que a aplicação está em modo de depuração, e quando você altera para o modo de lançamento, essa faixa não estará presente.

Mas essa faixa pode ser irritante para algumas pessoas, como eu, durante o desenvolvimento do aplicativo.

Para removê-la, basta definir o valor da propriedade **debugShowCheckedModeBanner** para **false** eu seu **MaterialApp**:

```dart
MaterialApp(
    debugShowCheckedModeBanner: false
)
```
Por exemplo, no modelo padrão:

```dart
MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
```
Pronto! A faixa de debug não será mais exibida na sua aplicação.

Se você preferir um vídeo, tenho este conteúdo disponível em um vídeo no YouTube:

{{< youtube S88yb0tsk9s >}}