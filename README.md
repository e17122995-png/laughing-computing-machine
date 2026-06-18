import telebot
import datetime
import os

# Замените 'YOUR_TOKEN' на токен от BotFather ИЛИ установите переменную окружения TELEGRAM_BOT_TOKEN
BOT_TOKEN = os.getenv('TELEGRAM_BOT_TOKEN', 'YOUR_TOKEN')

bot = telebot.TeleBot(BOT_TOKEN)

pending_users = {}
MIN_AGE = 16
BAN_DAYS = 365
DATE_FORMAT = '%d.%m.%Y'


@bot.message_handler(func=lambda message: True, content_types=['new_chat_members'])
def greet_new_user(message):
    for new_user in message.new_chat_members:
        if new_user.is_bot:
            continue
        user_id = new_user.id
        username = new_user.first_name or "Пользователь"
        pending_users[user_id] = {
            'username': username,
            'join_time': datetime.datetime.now(),
            'welcome_msg_id': None
        }
        welcome_text = (
            f"👋 Добро пожаловать, {username}!\n\n"
            "Для доступа в чат необходимо подтвердить возраст (16+).\n"
            "Отправьте дату рождения в формате ДД.ММ.ГГГГ (например, 01.01.2007):"
        )
        try:
            sent_message = bot.send_message(chat_id=message.chat.id, text=welcome_text)
            pending_users[user_id]['welcome_msg_id'] = sent_message.message_id
        except:
            pass

@bot.message_handler(content_types=['text'])
def handle_age_input(message):
    user_id = message.from_user.id
    if user_id not in pending_users:
        return
    try:
        birthdate = datetime.datetime.strptime(message.text.strip(), DATE_FORMAT).date()
        today = datetime.date.today()
        age = today.year - birthdate.year
        if (today.month, today.day) < (birthdate.month, birthdate.day):
            age -= 1
        if age >= MIN_AGE:
            try:
                bot.delete_message(chat_id=message.chat.id, message_id=message.message_id)
            except:
                pass
            success_text = f"✅ {pending_users[user_id]['username']}, ваш возраст подтверждён. Добро пожаловать в чат!"
            bot.send_message(chat_id=message.chat.id, text=success_text)
            del pending_users[user_id]
        else:
            until_date = int((datetime.datetime.now() + datetime.timedelta(days=BAN_DAYS)).timestamp())
            try:
                bot.ban_chat_member(
                    chat_id=message.chat.id,
                    user_id=user_id,
                    until_date=until_date
                )
                ban_text = (
                    f"❌ {pending_users[user_id]['username']}, доступ запрещён — требуется 16+.\n"
                    "Чат предназначен только для пользователей старше 16 лет."
                )
                bot.send_message(chat_id=message.chat.id, text=ban_text)
                del pending_users[user_id]
            except:
                pass
    except ValueError:
        bot.send_message(
            chat_id=message.chat.id,
            text="❌ Неверный формат даты. Отправьте дату в формате ДД.ММ.ГГГГ (например, 01.01.2007)."
        )
    except:
        bot.send_message(
            chat_id=message.chat.id,
            text="❌ Произошла ошибка. Попробуйте ещё раз или обратитесь к администратору."
        )

if name == "__main__":
    bot.polling(none_stop=True, interval=0)
