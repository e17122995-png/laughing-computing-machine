import logging
from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes

# Настройка логирования
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

# Токен вашего бота (замените на реальный)
BOT_TOKEN = "8808928786:AAGYJiDujkG226PBZ7F7dJGHtNQqnhhDKnQ"

async def ban_user(update: Update, context: ContextTypes.DEFAULT_TYPE):
    # Проверяем, что команда вызвана в группе/супергруппе
    if not update.message.chat.type in ['group', 'supergroup']:
        await update.message.reply_text("Эту команду можно использовать только в группах!")
        return

    # Проверяем права бота
    bot = context.bot
    bot_member
