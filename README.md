import requests
from telegram import Update
from telegram.ext import Updater, MessageHandler, Filters, CallbackContext
from googletrans import Translator

# === Replace with your actual bot token and API key ===
TOKEN = "7934434941:AAGUF7-FH0JXPQ0HkH95_MPQvTVbA-bD_uU"
API_KEY = "sk-fqOxk83Pj95U2AZxLsehy88qWjItr1G9"

translator = Translator()

def get_gpt_response(prompt, lang):
    url = "https://api.forefront.ai/v1/chat/completions"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {API_KEY}"
    }
    payload = {
        "model": "alpindale/Mistral-7B-v0.2-hf",
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": 150,
        "temperature": 0.7
    }

    try:
        res = requests.post(url, json=payload, headers=headers, timeout=10)
        if res.status_code == 200:
            reply = res.json().get("choices", [{}])[0].get("message", {}).get("content", "No response.")
        else:
            reply = f"Error from API: {res.status_code}"
    except Exception as e:
        reply = f"Connection error: {str(e)}"

    return translator.translate(reply, dest=lang).text if lang != "en" else reply

def handle_message(update: Update, context: CallbackContext):
    user_message = update.message.text
    user_lang = update.message.from_user.language_code

    if any(word in user_message.lower() for word in ["creator", "who made you", "who create you"]):
        reply = "My creator is Bruk Getachew (@Nameofbless), an Ethiopian developer. Please support him!"
    else:
        reply = get_gpt_response(user_message, user_lang)

    update.message.reply_text(reply)

def main():
    updater = Updater(TOKEN, use_context=True)
    dp = updater.dispatcher
    dp.add_handler(MessageHandler(Filters.text & ~Filters.command, handle_message))
    print("Bot is running...")
    updater.start_polling()
    updater.idle()

if __name__ == "__main__":
    main()
