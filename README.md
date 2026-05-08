# Atividade_1

import 'package:flutter/material.dart';
import 'package:permission_handler/permission_handler.dart';

void main() {
  runApp(const MaterialApp(home: TelaSimples()));
}

class TelaSimples extends StatefulWidget {
  const TelaSimples({super.key});

  @override
  State<TelaSimples> createState() => _TelaSimplesState();
}

class _TelaSimplesState extends State<TelaSimples> {
  String texto = 'Aguardando...';

  void solicitar() async {
    
    var status = await Permission.camera.request();
    
    setState(() {
      if (status.isGranted) {
        texto = 'Permissão concedida';
      } else {
        texto = 'Permissão negada';
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(texto),
            ElevatedButton(
              onPressed: solicitar,
              child: const Text('Pedir Câmera'),
            ),
          ],
        ),
      ),
    );
  }
}


2 - nivel dois , captura do acelerometro

import 'dart:async';
import 'package:flutter/material.dart';
import 'package:sensors_plus/sensors_plus.dart';

void main() {
  runApp(const MeuApp());
}

class MeuApp extends StatelessWidget {
  const MeuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: TelaAcelerometro(),
    );
  }
}

class TelaAcelerometro extends StatefulWidget {
  const TelaAcelerometro({super.key});

  @override
  State<TelaAcelerometro> createState() => _TelaAcelerometroState();
}

class _TelaAcelerometroState extends State<TelaAcelerometro> {
  double _x = 0.0;
  double _y = 0.0;
  double _z = 0.0;
  
  StreamSubscription<AccelerometerEvent>? _inscricao;

  @override
  void initState() {
    super.initState();
    _inscricao = accelerometerEventStream().listen((AccelerometerEvent evento) {
      setState(() {
        _x = evento.x;
        _y = evento.y;
        _z = evento.z;
      });
    });
  }

  @override
  void dispose() {
    _inscricao?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Nível 2 - Acelerômetro'),
        backgroundColor: Colors.green,
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Valores do Acelerômetro:',
              style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 30),
            // Mostrando os valores com apenas 2 casas decimais
            Text('Eixo X: ${_x.toStringAsFixed(2)}', style: const TextStyle(fontSize: 24)),
            Text('Eixo Y: ${_y.toStringAsFixed(2)}', style: const TextStyle(fontSize: 24)),
            Text('Eixo Z: ${_z.toStringAsFixed(2)}', style: const TextStyle(fontSize: 24)),
          ],
        ),
      ),
    );
  }
}


3 - nivel três , dectar movimento brusco

import 'dart:async';
import 'package:flutter/material.dart';
import 'package:sensors_plus/sensors_plus.dart';

void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: TelaMovimento(),
  ));
}

class TelaMovimento extends StatefulWidget {
  const TelaMovimento({super.key});

  @override
  State<TelaMovimento> createState() => _TelaMovimentoState();
}

class _TelaMovimentoState extends State<TelaMovimento> {
  String _mensagem = "Parado";
  StreamSubscription<AccelerometerEvent>? _inscricao;

  @override
  void initState() {
    super.initState();
    _inscricao = accelerometerEventStream().listen((AccelerometerEvent event) {
      setState(() {
        if (event.x > 8) {
          _mensagem = "Movimento detectado";
        } else {
          _mensagem = "Parado"; 
        }
      });
    });
  }

  @override
  void dispose() {
    _inscricao?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Nível 3 - Movimento Brusco'),
        backgroundColor: Colors.orange,
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Text(
          _mensagem,
          style: const TextStyle(fontSize: 30, fontWeight: FontWeight.bold),
        ),
      ),
    );
  }
}

4 - nivel quatro notificaçao local 

import 'dart:async';
import 'package:flutter/material.dart';
import 'package:sensors_plus/sensors_plus.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';


final FlutterLocalNotificationsPlugin pluginNotificacoes = FlutterLocalNotificationsPlugin();

void main() async {

  WidgetsFlutterBinding.ensureInitialized();

 
  const AndroidInitializationSettings configAndroid = AndroidInitializationSettings('@mipmap/ic_launcher');
  const InitializationSettings configGeral = InitializationSettings(android: configAndroid);
  
  
  await pluginNotificacoes.initialize(configGeral);

  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: TelaMovimentoNotificacao(),
  ));
}

class TelaMovimentoNotificacao extends StatefulWidget {
  const TelaMovimentoNotificacao({super.key});

  @override
  State<TelaMovimentoNotificacao> createState() => _TelaMovimentoNotificacaoState();
}

class _TelaMovimentoNotificacaoState extends State<TelaMovimentoNotificacao> {
  String _mensagem = "Parado";
  StreamSubscription<AccelerometerEvent>? _inscricao;
  
 
  bool _podeEnviarNotificacao = true;

  @override
  void initState() {
    super.initState();
    
   
    _inscricao = accelerometerEventStream().listen((AccelerometerEvent event) {
      setState(() {
        if (event.x > 8) {
          _mensagem = "Movimento detectado!";
          
          
          if (_podeEnviarNotificacao) {
            _enviarNotificacaoLocal();
            _bloquearNotificacoesTemporariamente(); 
          }
        } else {
          _mensagem = "Parado";
        }
      });
    });
  }

  
  Future<void> _enviarNotificacaoLocal() async {
    const AndroidNotificationDetails detalhesAndroid = AndroidNotificationDetails(
      'canal_movimento', 
      'Alertas de Movimento',  
      importance: Importance.max,
      priority: Priority.high,
    );
    const NotificationDetails detalhesGerais = NotificationDetails(android: detalhesAndroid);

   
    await pluginNotificacoes.show(
      0, 
      '🚨 Alerta!', 
      'Um movimento brusco foi detectado.', 
      detalhesGerais,
    );
  }

  
  void _bloquearNotificacoesTemporariamente() {
    _podeEnviarNotificacao = false; 
    Future.delayed(const Duration(seconds: 5), () {
      _podeEnviarNotificacao = true; 
    });
  }

  @override
  void dispose() {
    _inscricao?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Nível 4 - Notificação'),
        backgroundColor: Colors.redAccent,
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Text(
          _mensagem,
          style: const TextStyle(fontSize: 30, fontWeight: FontWeight.bold),
        ),
      ),
    );
  }
}


