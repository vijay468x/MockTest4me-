# MockTest4me-
For test
import logging
from telegram import InlineKeyboardButton, InlineKeyboardMarkup, Update
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    CallbackQueryHandler,
    ContextTypes,
)
import asyncio

# Enable logging to debug errors
logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO
)

# 1. Sample questions (can expand later)
questions = [
    {
        "question": "भारत का सबसे बड़ा राज्य कौन सा है?\nWhich is the largest state in India?",
        "options": ["A. राजस्थान", "B. मध्य प्रदेश", "C. उत्तर प्रदेश", "D. महाराष्ट्र"],
        "answer": "A. राजस्थान"
    },
    {
        "question": "भारतीय संविधान कब लागू हुआ?\nWhen was the Indian Constitution enacted?",
        "options": ["A. 26 जनवरी 1950", "B. 15 अगस्त 1947", "C. 26 नवंबर 1949", "D. 2 अक्टूबर 1950"],
        "answer": "A. 26 जनवरी 1950"
    }
]

user_index = {}

# 2. Generate keyboard
def build_keyboard(options):
    return InlineKeyboardMarkup([
        [InlineKeyboardButton(text=opt, callback_data=opt)] for opt in options
    ])

# 3. /start command
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    user_index[user_id] = 0  # Start from first question
    q = questions[0]
    await update.message.reply_text(q["question"], reply_markup=build_keyboard(q["options"]))

# 4. Button handling
async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    user_id = query.from_user.id
    index = user_index.get(user_id, 0)
    q = questions[index]

    selected = query.data
    correct = q["answer"]
    result = "✅ सही उत्तर!" if selected == correct else f"❌ गलत उत्तर!\nसही उत्तर: {correct}"

    # Show answer
    await query.edit_message_text(f"{q['question']}\n\nआपका उत्तर: {selected}\n\n{result}")

    # Wait & send next question
    await asyncio.sleep(2)
    index += 1
    if index < len(questions):
        user_index[user_id] = index
        next_q = questions[index]
        await query.message.reply_text(next_q["question"], reply_markup=build_keyboard(next_q["options"]))
    else:
        await query.message.reply_text("🎉 आपने सभी प्रश्न हल कर लिए!")

# 5. Run Bot
if __name__ == "__main__":
    try:
        app = ApplicationBuilder().token("7582875731:AAEGw4dPdH-3ziEZhkJGbDfULwOnzusM0Oc").build()

        app.add_handler(CommandHandler("start", start))
        app.add_handler(CallbackQueryHandler(button))

        print("🤖 Bot is running... don't stop the app")
        app.run_polling()
    except Exception as e:
        logging.error(f"Bot crashed: {e}")