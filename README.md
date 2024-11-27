# Лабораторная работа №6: Уведомления и напоминания в Android

- _Выполнил:_ Шестопалов Максим
- _Язык программирования:_ Kotlin <3

## Описание

Данное приложение на Android предоставляет пользователю возможность создавать напоминания с заголовком, текстом и датой, которые сохраняются в базу данных и отображаются в виде уведомлений. Напоминания можно просматривать, удалять, а также они отображаются в статус-баре с пользовательской иконкой. При нажатии на уведомление происходит переход к активности с полным описанием напоминания.

## Основные функции

1. **Создание напоминаний и сохранение в базу данных**: Пользователь может создать напоминание с указанием заголовка, текста и даты. Эти данные сохраняются в базу данных.

   ![0](https://github.com/user-attachments/assets/77c5f29c-a49a-4029-9851-72fec9ddc598)

   ```kotlin
   private fun saveReminder(title: String, message: String, reminderDate: Calendar) {
       // Сохранение в базу данных
       val reminder = Reminder(title, message, reminderDate.timeInMillis)
       reminderDao.insert(reminder)

       // Установка уведомления через AlarmManager
       setReminderAlarm(reminderDate.timeInMillis, title, message)
   }
   ```
   - Вводимые пользователем данные сохраняются в базу данных через `Room`.
   - После сохранения напоминания устанавливается уведомление с помощью `AlarmManager`.

3. **Просмотр напоминаний**: На отдельном экране отображается список всех созданных напоминаний.


![0](https://github.com/user-attachments/assets/fe01dcd1-7de0-4b28-8f87-abb348ce3907)


   ```kotlin
   private fun loadReminders() {
       val reminders = reminderDao.getAllReminders()
       reminderAdapter.submitList(reminders)
   }
   ```
   - Этот метод загружает напоминания из базы данных и отображает их в `RecyclerView`.

5. **Удаление напоминаний**: Пользователь может удалить любое напоминание, нажав на него в списке всех напоминаний.
   ```kotlin
   private fun deleteReminder(reminder: Reminder) {
       reminderDao.delete(reminder)
       loadReminders()  // Обновление списка после удаления
   }
   ```
   - Удаление происходит из базы данных, а список обновляется после удаления.

6. **Установка уведомления с помощью AlarmManager**: Уведомление для напоминания настраивается с использованием `AlarmManager`, `PendingIntent` и `BroadcastReceiver`.
   ```kotlin
   private fun setReminderAlarm(timeInMillis: Long, title: String, message: String) {
       val intent = Intent(this, ReminderBroadcastReceiver::class.java).apply {
           putExtra("title", title)
           putExtra("message", message)
       }

       val pendingIntent = PendingIntent.getBroadcast(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT)
       val alarmManager = getSystemService(Context.ALARM_SERVICE) as AlarmManager
       alarmManager.setExact(AlarmManager.RTC_WAKEUP, timeInMillis, pendingIntent)
   }
   ```
   - Этот код устанавливает точное время для уведомления, которое сработает в заданное время напоминания.

7. **Уведомления с переходом в активность**: При нажатии на уведомление открывается экран с полной информацией о напоминании


![image](https://github.com/user-attachments/assets/8a869fa5-6795-4bf3-953a-e0ea5ff7d249)

![0](https://github.com/user-attachments/assets/c06f7172-5236-4adf-9e24-36a7bd2cee72)



   ```kotlin
   class ReminderBroadcastReceiver : BroadcastReceiver() {
       override fun onReceive(context: Context, intent: Intent) {
           val title = intent.getStringExtra("title")
           val message = intent.getStringExtra("message")

           val notificationIntent = Intent(context, ReminderDetailActivity::class.java).apply {
               flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
               putExtra("title", title)
               putExtra("message", message)
           }

           val pendingIntent = PendingIntent.getActivity(context, 0, notificationIntent, PendingIntent.FLAG_UPDATE_CURRENT)

           val notification = NotificationCompat.Builder(context, "reminderChannel")
               .setContentTitle(title)
               .setContentText(message)
               .setSmallIcon(R.drawable.ic_notification)
               .setContentIntent(pendingIntent)
               .setAutoCancel(true)
               .build()

           val notificationManager = NotificationManagerCompat.from(context)
           notificationManager.notify(1, notification)
       }
   }
   ```
   - Этот код обрабатывает уведомление и обеспечивает переход в `ReminderDetailActivity`, когда пользователь нажимает на уведомление.

## Структура приложения

Приложение состоит из нескольких основных экранов и классов для работы с уведомлениями и базой данных:

1. **MainActivity**:
   - **Описание**: Это основной экран приложения, где пользователь может вводить заголовок, текст напоминания и выбирать дату/время.
   - **Кнопки**:
     - **Сохранить напоминание**: Добавляет новое напоминание в базу данных и устанавливает уведомление.
     - **Просмотреть напоминания**: Переход на экран со списком всех созданных напоминаний.

2. **ReminderListActivity**:
   - **Описание**: Этот экран отображает список всех напоминаний из базы данных.
   - **Элементы**:
     - **RecyclerView**: Список напоминаний, где каждое напоминание содержит заголовок, дату и время. Кнопка для удаления напоминания.

3. **ReminderDetailActivity**:
   - **Описание**: Этот экран показывает полную информацию о напоминании, когда пользователь нажимает на уведомление в статус-баре.
   - **Элементы**:
     - **TextView**: Отображение заголовка и текста напоминания.

4. **BroadcastReceiver**:
   - **Описание**: Отвечает за срабатывание уведомления в нужное время и обработку перехода на экран с деталями напоминания.

## Внешний вид приложения

- **MainActivity**:
  - Поля для ввода заголовка и сообщения напоминания.
  - Кнопка выбора даты и времени напоминания.
  - Кнопки для сохранения напоминания и просмотра списка напоминаний.

- **ReminderListActivity**: 
  - Список напоминаний с заголовком, датой, временем и кнопкой удаления.
![0](https://github.com/user-attachments/assets/df350ac8-5940-4a07-8336-7fe045e7e2a1)

![0](https://github.com/user-attachments/assets/df59a2af-fdc3-4011-bbfe-92ebaa4cdd01)


---

Приложение демонстрирует работу с уведомлениями и напоминаниями в Android, используя базу данных для хранения данных, `AlarmManager` для установки уведомлений и `BroadcastReceiver` для их обработки.

