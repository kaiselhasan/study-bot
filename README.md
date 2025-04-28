import os
import openai
import telebot
from telebot import types

# تلقي توكن بوت التليجرام من البيئة
TOKEN = os.getenv("TELEGRAM_TOKEN")
bot = telebot.TeleBot(TOKEN)

# إعداد API OpenAI
openai.api_key = os.getenv("OPENAI_API_KEY")

# دالة لتحويل المحاضرة إلى أسئلة
def generate_questions(text):
    prompt = f"حول هذا النص إلى 10 أسئلة اختيار من متعدد و 10 أسئلة صح وخطأ و 10 أسئلة مفتوحة و 10 أسئلة تعداد:\n\n{text}"
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=prompt,
        max_tokens=500
    )
    return response.choices[0].text.strip()

# التعامل مع الرسائل النصية
@bot.message_handler(func=lambda message: True)
def handle_message(message):
    if message.text:
        text = message.text
        questions = generate_questions(text)
        
        # إرسال الأسئلة إلى المستخدم
        bot.send_message(message.chat.id, f"الأسئلة التي تم توليدها:\n{questions}")
    else:
        bot.send_message(message.chat.id, "يرجى إرسال نصوص فقط لتحويلها إلى أسئلة.")

# التعامل مع الصور والملفات
@bot.message_handler(content_types=['photo', 'document'])
def handle_media(message):
    file_info = bot.get_file(message.photo[-1].file_id if message.content_type == 'photo' else message.document.file_id)
    downloaded_file = bot.download_file(file_info.file_path)

    # حفظ الملف
    file_name = f"received_file_{message.message_id}.jpg" if message.content_type == 'photo' else f"received_file_{message.message_id}.pdf"
    with open(file_name, 'wb') as new_file:
        new_file.write(downloaded_file)

    bot.send_message(message.chat.id, f"تم استلام الملف: {file_name}. سيتم معالجته قريبًا.")

# تشغيل البوت
bot.polling(none_stop=True)
