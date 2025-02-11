# Telegram_Bot
import telebot
from telebot import types

TOKEN = '7942445181:AAEv5cSCrIvHiSe91_Arbsr5BEh5LyC0ZqU'  # توکن جدید
ADMIN_ID = '6816923182'  # آیدی عددی شما
CHANNEL_USERNAME = '@Cliniic_Behnood'  # یوزرنیم کانال

bot = telebot.TeleBot(TOKEN)

reserved_times = {
    'ارتوپدی': [],
    'داخلی': [],
    'زنان و زایمان': [],
    'قلب و عروق': [],
    'اطفال': []
}

weekly_schedule = """
📅 *برنامه هفتگی درمانگاه بهنود*:
- 🦴 *ارتوپدی:* روزهای زوج، 16:30 تا 18:30
- 🏥 *داخلی:* دوشنبه‌ها، 14:30 تا 19
- 👩‍⚕ *زنان و زایمان:* یکشنبه‌ها، 10 تا 13
- ❤️ *قلب و عروق:* یکشنبه‌ها، 16 تا 18
- 👶 *اطفال:* فعلاً پزشکی نداریم
"""

def check_channel_member(chat_id):
    try:
        member = bot.get_chat_member(CHANNEL_USERNAME, chat_id)
        return member.status in ['member', 'administrator', 'creator']
    except Exception as e:
        return False

@bot.message_handler(commands=['start'])
def start(message):
    chat_id = message.chat.id
    if check_channel_member(chat_id):
        bot.send_message(chat_id, f"سلام {message.chat.first_name} عزیز! 🌟\n{weekly_schedule}\n\nلطفاً تخصص مورد نظر خود را انتخاب کنید.", parse_mode="Markdown")
        show_specialties(message)
    else:
        bot.send_message(chat_id, f"🔴 برای استفاده از ربات، ابتدا باید عضو کانال باشید.\n\n📢 [عضویت در کانال]({CHANNEL_USERNAME})", parse_mode="Markdown", disable_web_page_preview=True)
        bot.send_message(chat_id, "✅ پس از عضویت، دوباره /start را بزنید.")

def show_specialties(message):
    markup = types.ReplyKeyboardMarkup(row_width=2, resize_keyboard=True)
    markup.add("ارتوپدی", "داخلی", "زنان و زایمان", "قلب و عروق", "اطفال")
    bot.send_message(message.chat.id, "🔹 لطفاً تخصص مورد نظر را انتخاب کنید:", reply_markup=markup)

@bot.message_handler(func=lambda message: message.text in ["ارتوپدی", "داخلی", "زنان و زایمان", "قلب و عروق", "اطفال"])
def handle_specialty(message):
    specialty = message.text
    if specialty not in reserved_times:
        bot.send_message(message.chat.id, "❌ تخصص مورد نظر در حال حاضر در دسترس نیست.")
    else:
        if len(reserved_times[specialty]) < 5:  # فرض کنیم که حداکثر 5 نفر برای هر تخصص ثبت نام می‌کنند
            reserved_times[specialty].append(message.chat.id)
            bot.send_message(message.chat.id, f"✅ شما برای تخصص *{specialty}* ثبت‌نام شدید.\n\nزمان نوبت شما: {weekly_schedule}")
        else:
            bot.send_message(message.chat.id, "❌ ظرفیت برای این تخصص پر شده است.")
