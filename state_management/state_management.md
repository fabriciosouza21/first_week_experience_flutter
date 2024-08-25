## O que sera visto

- Statefulwidget
- compartilahndo esdo entre widgets usandos construtores, InheritedWidget
- Usando Listanable para notifica outros widgets
- Usando Model-View-ViewModel (MVVM) para arquitetura do seu aplicativo

## StatefulWidget

A maneira mais simples de gerenciar o estado é usar um StatefulWidget, que armazena o estado dentro de im mesmo.


```dart
class MyCounter extends StatefulWidget {
  const MyCounter({super.key});

  @override
  State<MyCounter> createState() => _MyCounterState();
}

class _MyCounterState extends State<MyCounter> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $count'),
        TextButton(
          onPressed: () {
            setState(() {
              count++;
            });
          },
          child: Text('Increment'),
        )
      ],
    );
  }
}
```


- Encpsulamento : o Widget MyCounter não tem visivbilidade da count variável subjacente e não tem meios de acessá-la ou alterá-la.
- Ciclo de vida: O _MyCounterState objeto e sua count variável são criados na primeira vez que MyCounter são construídos e existem até serem removidos da tela


Estado efêmero (às vezes chamado de estado da IU ou estado local ) é o estado que você pode conter perfeitamente em um único widget.

[Estado efêmero e estado do aplicativo](https://docs.flutter.dev/data-and-backend/state-mgmt/ephemeral-vs-app)


## Compartilhando estado entre widgets

cenários em que um aplicativo precisa armazena estado incluem o seguinte:
- para atualizar o estado
- para ouvir

esta seção explora como vocẽ pode efetivamente compartilha o estado entre diferentes widgets:
- usando construtores
- usando InheritedWidget
- usando retorno de chamada

### Usando construtores
Os objetos dart são passados por referênci, é muito comum que os widgets definam os objetos que precisam usar em seu construtor.

``` dart
class MyCounter extends StatelessWidget {
  final int count;
  const MyCounter({super.key, required this.count});

  @override
  Widget build(BuildContext context) {
    return Text('$count');
  }
}

Column(
  children: [
    MyCounter(
      count: count,
    ),
    MyCounter(
      count: count,
    ),
    TextButton(
      child: Text('Increment'),
      onPressed: () {
        setState(() {
          count++;
        });
      },
    )
  ],
)
```

essa forma de gerencia estado utilizar a injenção de dependência, que é um padrão de design que permite que um objeto forneça as dependências de outro objeto. Fica claro que o MyCounter depende de um count, e o count é fornecido pelo widget pai.


### Usando InheritedWidget

``` dart

class MyState extends InheritedWidget {
  const MyState({
    super.key,
    required this.data,
    required super.child,
  });

  final String data;

  static MyState of(BuildContext context) {
    // This method looks for the nearest `MyState` widget ancestor.
    final result = context.dependOnInheritedWidgetOfExactType<MyState>();

    assert(result != null, 'No MyState found in context');

    return result!;
  }

  @override
  // This method should return true if the old widget's data is different
  // from this widget's data. If true, any widgets that depend on this widget
  // by calling `of()` will be re-built.
  bool updateShouldNotify(MyState oldWidget) => data != oldWidget.data;
}

// Em seguida, chame o of()método do build()método do widget que precisa de acesso ao estado compartilhado:

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    var data = MyState.of(context).data;
    return Scaffold(
      body: Center(
        child: Text(data),
      ),
    );
  }
}
```
## Usando Callbacks

``` dart
// Define a callback signature
typedef ValueChanged<T> = void Function(T value);

class MyCounter extends StatefulWidget {
  const MyCounter({super.key, required this.onChanged});

  final ValueChanged<int> onChanged;

  @override
  State<MyCounter> createState() => _MyCounterState();
}

TextButton(
  onPressed: () {
    widget.onChanged(count++);
  },
),


```

## Usando Listenable

- ChangeNotifier
- ValueNotifier

### ChangeNotifier

``` dart

class CounterNotifier extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}

//Em seguida, passe-o para ListenableBuilderpara garantir que a subárvore retornada pela builderfunção seja reconstruída sempre que ela ChangeNotifieratualizar seus ouvintes.
Column(
  children: [
    ListenableBuilder(
      listenable: counterNotifier,
      builder: (context, child) {
        return Text('counter: ${counterNotifier.count}');
      },
    ),
    TextButton(
      child: Text('Increment'),
      onPressed: () {
        counterNotifier.increment();
      },
    ),
  ],
)

```

CounterNotifier é um ChangeNotifier que mantém um contador e notifica seus ouvintes sempre que o contador é incrementado. O ListenableBuilder widget é um widget que reconstrói sua subárvore sempre que o Listenable fornecido notifica seus ouvintes. o valor do contador é alterado, o CounterNotifier.increment() método é chamado, que incrementa o contador e notifica seus ouvintes. O ListenableBuilder widget reconstrói a subárvore e exibe o novo valor do contador.


### ValueNotifier

A ValueNotifier é ver~sao mais simples de a ChangeNotifier, que armazena um único valor.


``` dart

ValueNotifier<int> counterNotifier = ValueNotifier(0);

Column(
  children: [
    ValueListenableBuilder(
      valueListenable: counterNotifier,
      builder: (context, child, value) {
        return Text('counter: $value');
      },
    ),
    TextButton(
      child: Text('Increment'),
      onPressed: () {
        counterNotifier.value++;
      },
    ),
  ],
)

```

funcionar similar ao ChangeNotifier, mas com um valor único. O ValueListenableBuilder widget é um widget que reconstrói sua subárvore sempre que o ValueListenable fornecido notifica seus ouvintes. O valor do contador é alterado, o counterNotifier.value++ é chamado, que incrementa o contador e notifica seus ouvintes. O ValueListenableBuilder widget reconstrói a subárvore e exibe o novo valor do contador.



## Usando Model-View-ViewModel (MVVM)

``` dart

import 'package:http/http.dart';

class CounterData {
  CounterData(this.count);

  final int count;
}

class CounterModel {
  Future<CounterData> loadCountFromServer() async {
    final uri = Uri.parse('https://myfluttercounterapp.net/count');
    final response = await get(uri);

    if (response.statusCode != 200) {
      throw ('Failed to update resource');
    }

    return CounterData(int.parse(response.body));
  }

  Future<CounterData> updateCountOnServer(int newCount) async {
    // ...
  }
}

```

Este modelo não usa nenhuma primitiva Flutter ou faz suposições sobre a plataforma em que está sendo executado; seu único trabalho é buscar ou atualizar a contagem usando seu cliente HTTP. Isso permite que o modelo seja implementado com um Mock ou Fake em testes unitários e define limites claros entre os componentes de baixo nível do seu aplicativo e os componentes de UI de nível mais alto necessários para construir o aplicativo completo.


### Definido o viewModel

``` dart
import 'package:flutter/foundation.dart';

class CounterViewModel extends ChangeNotifier {
  final CounterModel model;
  int? count;
  String? errorMessage;
  CounterViewModel(this.model);

  Future<void> init() async {
    try {
      count = (await model.loadCountFromServer()).count;
    } catch (e) {
      errorMessage = 'Could not initialize counter';
    }
    notifyListeners();
  }

  Future<void> increment() async {
    var count = this.count;
    if (count == null) {
      throw('Not initialized');
    }
    try {
      await model.updateCountOnServer(count + 1);
      count++;
    } catch(e) {
      errorMessage = 'Count not update count';
    }
    notifyListeners();
  }
}
```

### Definindo a View

``` dart

ListenableBuilder(
  listenable: viewModel,
  builder: (context, child) {
    return Column(
      children: [
        if (viewModel.errorMessage != null)
          Text(
            'Error: ${viewModel.errorMessage}',
            style: Theme.of(context)
                .textTheme
                .labelSmall
                ?.apply(color: Colors.red),
          ),
        Text('Count: ${viewModel.count}'),
        TextButton(
          onPressed: () {
            viewModel.increment();
          },
          child: Text('Increment'),
        ),
      ],
    );
  },
)

```

No exemplo acima ele faz a separação o model, não tem qual quer dependência do flutter, o viewModel é responsável por gerenciar o estado e a view é responsável por exibir o estado. é uma forma de separar a lógica de negócios da lógica de apresentação.


## Provedor

Um wrapper em torno do InheritedWidget para torná-lo mais fácil de usar e mais reutilizável.

Ao usar InheritedWidgetprovider em vez de escrever manualmente , você obtém:

alocação/disposição simplificada de recursos
carregamento lento
um clichê bastante reduzido sobre a criação de uma nova classe toda vez
devtool amigável – usando o Provider, o estado do seu aplicativo ficará visível no Flutter devtool
uma maneira comum de consumir esses InheritedWidget s (consulte Provider.of / Consumer / Selector )
maior escalabilidade para classes com um mecanismo de escuta que cresce exponencialmente em complexidade (como ChangeNotifier , que é O(N) para despachar notificações).

``` dart
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

/// This is a reimplementation of the default Flutter application using provider + [ChangeNotifier].

void main() {
  runApp(
    /// Providers are above [MyApp] instead of inside it, so that tests
    /// can use [MyApp] while mocking the providers
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => Counter()),
      ],
      child: const MyApp(),
    ),
  );
}

/// Mix-in [DiagnosticableTreeMixin] to have access to [debugFillProperties] for the devtool
// ignore: prefer_mixin
class Counter with ChangeNotifier, DiagnosticableTreeMixin {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }

  /// Makes `Counter` readable inside the devtools by listing all of its properties
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(IntProperty('count', count));
  }
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Example'),
      ),
      body: const Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('You have pushed the button this many times:'),

            /// Extracted as a separate widget for performance optimization.
            /// As a separate widget, it will rebuild independently from [MyHomePage].
            ///
            /// This is totally optional (and rarely needed).
            /// Similarly, we could also use [Consumer] or [Selector].
            Count(),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        key: const Key('increment_floatingActionButton'),

        /// Calls `context.read` instead of `context.watch` so that it does not rebuild
        /// when [Counter] changes.
        onPressed: () => context.read<Counter>().increment(),
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}

class Count extends StatelessWidget {
  const Count({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Text(
      /// Calls `context.watch` to make [Count] rebuild when [Counter] changes.
      '${context.watch<Counter>().count}',
      key: const Key('counterState'),
      style: Theme.of(context).textTheme.headlineMedium,
    );
  }
}
```

``` dart

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

/// This is a reimplementation of the default Flutter application using provider + [ChangeNotifier].

void main() {
  runApp(
    /// Providers are above [MyApp] instead of inside it, so that tests
    /// can use [MyApp] while mocking the providers
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => Counter()),
      ],
      child: const MyApp(),
    ),
  );
}

/// Mix-in [DiagnosticableTreeMixin] to have access to [debugFillProperties] for the devtool
// ignore: prefer_mixin
class Counter with ChangeNotifier, DiagnosticableTreeMixin {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }

  /// Makes `Counter` readable inside the devtools by listing all of its properties
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.add(IntProperty('count', count));
  }
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Example'),
      ),
      body: const Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('You have pushed the button this many times:'),

            /// Extracted as a separate widget for performance optimization.
            /// As a separate widget, it will rebuild independently from [MyHomePage].
            ///
            /// This is totally optional (and rarely needed).
            /// Similarly, we could also use [Consumer] or [Selector].
            Count(),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        key: const Key('increment_floatingActionButton'),

        /// Calls `context.read` instead of `context.watch` so that it does not rebuild
        /// when [Counter] changes.
        onPressed: () => context.read<Counter>().increment(),
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}

class Count extends StatelessWidget {
  const Count({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Text(
      /// Calls `context.watch` to make [Count] rebuild when [Counter] changes.
      '${context.watch<Counter>().count}',
      key: const Key('counterState'),
      style: Theme.of(context).textTheme.headlineMedium,
    );
  }
}
```

Usar o `Consumer` no Flutter, especialmente quando se está trabalhando com o pacote `provider`, oferece alguns benefícios importantes em comparação ao uso direto de `watch` dentro de widgets.

### Por que usar `Consumer` em vez de `watch` diretamente?

1. **Rebuild Control**: Com o `Consumer`, você tem controle mais granular sobre o que é reconstruído. Você pode colocar o `Consumer` apenas ao redor do widget que precisa ser atualizado, o que minimiza a quantidade de widgets que são reconstruídos quando os dados mudam.

2. **Melhor Performance**: Como o `Consumer` permite reconstruir apenas a parte da árvore de widgets que depende de um `provider`, isso pode melhorar a performance da sua aplicação ao evitar reconstruções desnecessárias de widgets.

3. **Separa a Lógica de Negócio da UI**: Usando `Consumer`, você separa melhor a lógica de atualização de estado da lógica de renderização da interface do usuário. Isso mantém seu código mais organizado e facilita a manutenção.

4. **Facilidade de leitura**: O código usando `Consumer` pode ser mais fácil de entender e manter, pois deixa claro quais partes da interface do usuário dependem de quais partes do estado.

### Exemplo utilizando `Consumer` no Flutter

Vamos considerar um exemplo simples onde temos um contador que atualiza um texto na tela:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => Counter(),
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: CounterScreen(),
    );
  }
}

class CounterScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Contador'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('Você pressionou o botão esta muitas vezes:'),
            Consumer<Counter>(
              builder: (context, counter, child) {
                return Text(
                  '${counter.value}',
                  style: Theme.of(context).textTheme.headlineMedium,
                );
              },
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          Provider.of<Counter>(context, listen: false).increment();
        },
        tooltip: 'Incrementar',
        child: Icon(Icons.add),
      ),
    );
  }
}

class Counter with ChangeNotifier {
  int _value = 0;

  int get value => _value;

  void increment() {
    _value++;
    notifyListeners();
  }
}
```

### Análise do Exemplo

1. **Provider Setup**: Usamos `ChangeNotifierProvider` para criar uma instância do `Counter` e torná-lo disponível para a árvore de widgets.

2. **Uso do `Consumer`**: No widget `CounterScreen`, o `Consumer` é usado para escutar mudanças no `Counter` e reconstruir apenas o `Text` que mostra o valor atual do contador.

3. **Atualização de Estado**: O botão de incremento usa `Provider.of<Counter>(context, listen: false)` para acessar o `Counter` e chamar o método `increment` sem escutar mudanças, pois o botão em si não precisa ser reconstruído quando o contador muda.

Essa abordagem com `Consumer` permite que apenas o texto do contador seja reconstruído quando o valor muda, evitando reconstruções desnecessárias de outros widgets.

