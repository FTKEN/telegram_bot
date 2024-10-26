import os
import telebot
from pytube import YouTube
from instaloader import Instaloader, Post
from tiktok_downloader import snaptik  # TikTok uchun snaptik moduli ishlatish mumkin

bot = telebot.TeleBot("YOUR_BOT_TOKEN")  # O'z tokeningizni joylang

@bot.message_handler(content_types=['text'])
def handle_text(message):
    try:
        url = message.text.strip()
        platform = detect_platform(url)

        if platform == "youtube":
            download_youtube(url, message.chat.id)
        elif platform == "instagram":
            download_instagram(url, message.chat.id)
        elif platform == "tiktok":
            download_tiktok(url, message.chat.id)
        else:
            bot.send_message(message.chat.id,
                             "Unsupported platform. Please send a valid YouTube, Instagram, or TikTok link.")
    except Exception as e:
        bot.send_message(message.chat.id, f"Error: {e}")

def detect_platform(url):
    """ URL orqali qaysi platforma ekanligini aniqlash """
    if "youtube.com" in url or "youtu.be" in url:
        return "youtube"
    elif "instagram.com" in url:
        return "instagram"
    elif "tiktok.com" in url:
        return "tiktok"
    else:
        return "unknown"

def download_youtube(url, chat_id):
    """ YouTube videolarini yuklab olish va foydalanuvchiga jo'natish """
    try:
        yt = YouTube(url)
        stream = yt.streams.filter(progressive=True, file_extension='mp4').order_by('resolution').desc().first()
        stream.download(filename="downloaded_video.mp4")

        with open("downloaded_video.mp4", "rb") as f:
            bot.send_document(chat_id, f)

        os.remove("downloaded_video.mp4")
    except Exception as e:
        bot.send_message(chat_id, f"Error downloading YouTube video: {e}")

def download_instagram(url, chat_id):
    """ Instagram videolarini yoki postlarini yuklab olish va foydalanuvchiga jo'natish """
    try:
        L = Instaloader()
        shortcode = url.split("/")[-2]  # Instagram postining shortcode qismini olish
        post = Post.from_shortcode(L.context, shortcode)

        # Video mavjudligini tekshirish
        if post.is_video:
            L.download_post(post, target="downloads")

            # Yuklangan faylni foydalanuvchiga yuborish
            video_filename = f"downloads/{post.shortcode}.mp4"
            with open(video_filename, "rb") as f:
                bot.send_document(chat_id, f)

            # Yuklangan fayllarni o'chirish
            os.remove(video_filename)
        else:
            bot.send_message(chat_id, "Bu postda video yo'q.")
    except Exception as e:
        bot.send_message(chat_id, f"Error downloading Instagram video: {e}")

def download_tiktok(url, chat_id):
    """ TikTok videolarini yuklab olish va foydalanuvchiga jo'natish """
    try:
        video_data = snaptik(url)  # TikTok uchun `snaptik` dan foydalanish

        # Yuklangan TikTok videoni saqlash
        with open('downloaded_tiktok_video.mp4', 'wb') as f:
            f.write(video_data)

        with open('downloaded_tiktok_video.mp4', 'rb') as f:
            bot.send_document(chat_id, f)

        os.remove('downloaded_tiktok_video.mp4')
    except Exception as e:
        bot.send_message(chat_id, f"Error downloading TikTok video: {e}")

bot.infinity_polling()
