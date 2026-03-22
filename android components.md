В андроид есть 4 основных компонента приложения: activity, service, contentProvider, BroadcastReceivier

## BroadcastReceivier

Позволяют слушать события системы или других приложений.
Есть 2 типа - статические и динамические.
Статические рисиверы описываются в файле манифеста и могут разбить приложение. Но начиная с 8 андроих их сильно урезали и многие системные события нужно слушать с поомщью динамического ресивера
Динамический ресивер - также позволяет слушать системные события, но ограничен жизненным циклом вызвавшего компонента. 
Регистрация происходит через Context#registerReceiver, а отмена регистрации через Context#unregisterReceiver

Для передачи сообщений внутри приложения использоваться LocalBroadcastManager, который признан устаревшим. Пример его использования: 
```kotlin
// Регистрация
LocalBroadcastManager.getInstance(context)
    .registerReceiver(myReceiver, IntentFilter("my-custom-action"))

// Отправка сообщения
val intent = Intent("my-custom-action")
intent.putExtra("data", "Hello Internal!")
LocalBroadcastManager.getInstance(context).sendBroadcast(intent)

// Отмена
LocalBroadcastManager.getInstance(context).unregisterReceiver(myReceiver)
```

