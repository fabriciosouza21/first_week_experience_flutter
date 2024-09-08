# Handling user input

https://api.flutter.dev/flutter/material/CheckboxListTile-class.html?_gl=1*unw3sb*_gcl_aw*R0NMLjE3MjM3MzAyODEuQ2owS0NRand6dmExQmhEM0FSSXNBRFF1UG5VVm9JYmNHRnhhQU9kN1RSZnMzQVRxd29URWhaR1JMRVFWTF8tZXFnYV9LTzNJeDhtcDhLRWFBbWhFRUFMd193Y0I.*_gcl_dc*R0NMLjE3MjM3MzAyODEuQ2owS0NRand6dmExQmhEM0FSSXNBRFF1UG5VVm9JYmNHRnhhQU9kN1RSZnMzQVRxd29URWhaR1JMRVFWTF8tZXFnYV9LTzNJeDhtcDhLRWFBbWhFRUFMd193Y0I.*_ga*NjY0NDg4ODE5LjE3MjI5NDYwODk.*_ga_04YGWK0175*MTcyNTcxMjk4NC4xMC4xLjE3MjU3MTQwMzcuMC4wLjA.

## Widgets botões

- ElevatedButton
- FillButton
- OutlineButton
- TextButton
- IconButton
- FloatingActionButton

##  Widgest suportam entrada de texto

## textfield

Permita que os usuários insiram texto, seja com o teclado físico ou com um teclado na tela.
A série de artigos de receitas a seguir explica cada etapa de como criar um campo de texto, recuperar seu valor e lidar com alterações de estado:

Artigo: [Criar e estilizar um campo de texto](https://docs.flutter.dev/cookbook/forms/text-input)
Artigo: [Recuperar o valor de um campo de texto](https://docs.flutter.dev/cookbook/forms/retrieve-input)
Artigo: [Lidar com alterações em um campo de texto](https://docs.flutter.dev/cookbook/forms/text-field-changes)
Artigo: [Campos de foco e texto ](https://docs.flutter.dev/cookbook/forms/focus)

## RichText

Demonstração: [Editor de Rich Text](https://flutter.github.io/samples/rich_text_editor.html)
Código de exemplo: código do editor de texto avançado

## SelectableText

- [SelectableText (Widget da Semana) ](https://www.youtube.com/watch?v=ZSU3ZXOs6hc)

## Form

[Artigo: Crie um formulário com validação](https://docs.flutter.dev/cookbook/forms/validation)
[Demonstração: Aplicativo de formulário](https://flutter.github.io/samples/web/form_app/)
[Código de exemplo: código do aplicativo de formulário](https://github.com/flutter/samples/tree/main/form_app)

## Selecione um valor de um grupo de opções

- SegmentedButton
- Slider
- DropdownMenu
- Slider

## Alternar valores

- Checkbox
- CheckboxListTile
- Switch
- SwitchListTile
- Radio
- Chip

## Data ou hora

- showDatePicker
- showTimePicker

## Deslize e deslize

[Dismissible (Widget da Semana)](https://docs.flutter.dev/get-started/fwe/user-input#:~:text=Dismissible%20(Widget%20da%20Semana))
[Artigo: Implementar deslizar para dispensar](https://docs.flutter.dev/cookbook/gestures/dismissible)


 ``` dart
import 'package:flutter/material.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    const appTitle = 'Form Validation Demo';

    return MaterialApp(
      title: appTitle,
      home: Scaffold(
        appBar: AppBar(
          title: const Text(appTitle),
        ),
        body: const MyCustomForm(),
      ),
    );
  }
}

// Create a Form widget.
class MyCustomForm extends StatefulWidget {
  const MyCustomForm({super.key});

  @override
  MyCustomFormState createState() {
    return MyCustomFormState();
  }
}

// Create a corresponding State class.
// This class holds data related to the form.
class MyCustomFormState extends State<MyCustomForm> {
  // Create a global key that uniquely identifies the Form widget
  // and allows validation of the form.
  //
  // Note: This is a GlobalKey<FormState>,
  // not a GlobalKey<MyCustomFormState>.
  final _formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    // Build a Form widget using the _formKey created above.
    return Form(
      key: _formKey,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          TextFormField(
            // The validator receives the text that the user has entered.
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Please enter some text';
              }
              return null;
            },
          ),
          Padding(
            padding: const EdgeInsets.symmetric(vertical: 16),
            child: ElevatedButton(
              onPressed: () {
                // Validate returns true if the form is valid, or false otherwise.
                if (_formKey.currentState!.validate()) {
                  // If the form is valid, display a snackbar. In the real world,
                  // you'd often call a server or save the information in a database.
                  ScaffoldMessenger.of(context).showSnackBar(
                    const SnackBar(content: Text('Processing Data')),
                  );
                }
              },
              child: const Text('Submit'),
            ),
          ),
        ],
      ),
    );
  }
}
```

