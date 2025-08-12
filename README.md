from aiogram import Bot, Dispatcher, executor, types
from aiogram.types import InlineKeyboardButton, InlineKeyboardMarkup
import asyncio
import os

TOKEN = os.getenv("BOT_TOKEN")  # Token Render sozlamalaridan olinadi
KANALLAR = ["@sevgimizoxirgacha", "@aslwinemuhlislari", "@LazizWaynee"]

bot = Bot(token=TOKEN)
dp = Dispatcher(bot)

# Kanal tugmalari
def kanal_tugmalari():
    klav = InlineKeyboardMarkup(row_width=1)
    for kanal in KANALLAR:
        klav.add(InlineKeyboardButton(f"📢 {kanal} kanaliga obuna bo‘lish", url=f"https://t.me/{kanal[1:]}"))
    klav.add(InlineKeyboardButton("✅ Obunani tekshirish", callback_data="tekshir"))
    return klav

# Obunani tekshirish
async def obuna_tekshir(user_id):
    for kanal in KANALLAR:
        try:
            status = await bot.get_chat_member(kanal, user_id)
            if status.status not in ["member", "administrator", "creator"]:
                return False
        except:
            return False
    return True

# /start komandasi
@dp.message_handler(commands=["start"])
async def start_komandasi(xabar: types.Message):
    if await obuna_tekshir(xabar.from_user.id):
        await xabar.answer("✅ Botga xush kelibsiz! Endi bot funksiyalaridan foydalanishingiz mumkin.")
    else:
        await xabar.answer(
            "🚫 Botdan foydalanish uchun quyidagi kanallarga obuna bo‘ling:",
            reply_markup=kanal_tugmalari()
        )

# Tugma bosilganda
@dp.callback_query_handler(lambda c: c.data == "tekshir")
async def tugma_tekshir(callback: types.CallbackQuery):
    if await obuna_tekshir(callback.from_user.id):
        await callback.message.edit_text("✅ Rahmat! Siz barcha kanallarga obuna bo‘ldingiz. Bot endi ishlaydi.")
    else:
        await callback.answer("❗ Hali barcha kanallarga obuna bo‘lmadingiz.", show_alert=True)

if __name__ == "__main__":
    executor.start_polling(dp, skip_updates=True)
