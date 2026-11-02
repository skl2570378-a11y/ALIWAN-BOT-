# ALIWAN-BOT-
# ALIWAN BOT - Telegram referral and points management bot. Developed by Aliwan 

import telebot
import sqlite3
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton

TOKEN = "PUT_YOUR_BOT_TOKEN_HERE"
BOT_USERNAME = "alioan2_bot"
BONUS = 100000

bot = telebot.TeleBot(TOKEN)

db = sqlite3.connect("aliwan.db", check_same_thread=False)
cursor = db.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    user_id INTEGER PRIMARY KEY,
    points INTEGER DEFAULT 0
)
""")
db.commit()


def add_user(user_id):
    cursor.execute(
        "INSERT OR IGNORE INTO users (user_id) VALUES (?)",
        (user_id,)
    )
    db.commit()


def get_points(user_id):
    cursor.execute(
        "SELECT points FROM users WHERE user_id=?",
        (user_id,)
    )
    result = cursor.fetchone()
    return result[0] if result else 0


def keyboard():
    markup = InlineKeyboardMarkup()
    markup.add(
        InlineKeyboardButton("💰 نقاطي", callback_data="points"),
        InlineKeyboardButton("🔗 رابط دعوتي", callback_data="invite")
    )
    return markup


@bot.message_handler(commands=["start"])
def start(message):
    user_id = message.from_user.id
    add_user(user_id)

    parts = message.text.split()

    if len(parts) > 1:
        try:
            referrer = int(parts[1])

            if referrer != user_id:
                cursor.execute(
                    "SELECT points FROM users WHERE user_id=?",
                    (referrer,)
                )
                if cursor.fetchone():
                    cursor.execute(
                        "UPDATE users SET points=points+? WHERE user_id=?",
                        (BONUS, user_id)
                    )
                    db.commit()
        except ValueError:
            pass

    bot.send_message(
        message.chat.id,
        "🟢 أهلاً بك في ALIWAN BOT\n\n"
        "🎁 مكافأة دخولك: 100,000 نقطة\n"
        "اختر من الأزرار:",
        reply_markup=keyboard()
    )


@bot.callback_query_handler(func=lambda call: True)
def buttons(call):
    user_id = call.from_user.id

    if call.data == "points":
        points = get_points(user_id)
        bot.answer_callback_query(call.id)
        bot.send_message(
            call.message.chat.id,
            f"💰 رصيدك الحالي: {points:,} نقطة"
        )

    elif call.data == "invite":
        link = f"https://t.me/{BOT_USERNAME}?start={user_id}"
        bot.answer_callback_query(call.id)
        bot.send_message(
            call.message.chat.id,
            f"🔗 رابط دعوتك الخاص:\n{link}"
        )


print("ALIWAN BOT STARTED 🟢")
bot.infinity_polling(skip_pending=True)
