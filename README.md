import time
import re
from telethon.sync import TelegramClient, events
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from webdriver_manager.chrome import ChromeDriverManager

# === Настройки Telegram ===
API_ID = 'ВАШ_API_ID'  # Вставьте свой API ID
API_HASH = 'ВАШ_API_HASH'  # Вставьте свой API Hash
GROUP_ID = -1001234567890  # ID группы Telegram (узнайте через @getmyid_bot)
PHONE_NUMBER = 'ВАШ_НОМЕР'  # Телефон для входа в Telegram

# === Настройки Binance ===
BINANCE_URL = "https://www.binance.com/ru/my/coupon"
USERNAME = "ВАШ_EMAIL"  # Логин от Binance
PASSWORD = "ВАШ_ПАРОЛЬ"  # Пароль от Binance

# === Настройка Selenium ===
options = webdriver.ChromeOptions()
options.add_argument("--headless")  # Запуск без интерфейса (можно убрать)
options.add_argument("--disable-gpu")
options.add_argument("--no-sandbox")

driver = webdriver.Chrome(ChromeDriverManager().install())

# === Функция ввода кода на Binance ===
def enter_binance_code(code):
    driver.get(BINANCE_URL)
    time.sleep(5)

    # Вход в аккаунт (если требуется)
    if "login" in driver.current_url:
        login_field = driver.find_element(By.NAME, "email")
        password_field = driver.find_element(By.NAME, "password")

        login_field.send_keys(USERNAME)
        password_field.send_keys(PASSWORD)
        password_field.send_keys(Keys.RETURN)

        time.sleep(5)

    # Вставка кода
    try:
        input_box = driver.find_element(By.NAME, "redeemCode")  # Убедись, что поле называется так
        input_box.send_keys(code)
        input_box.send_keys(Keys.RETURN)
        print(f"Код {code} успешно введён на Binance!")
    except Exception as e:
        print(f"Ошибка при вводе кода: {e}")

# === Функция обработки сообщений Telegram ===
async def handle_new_message(event):
    message = event.message.message
    match = re.search(r"[A-Z0-9]{16,}", message)  # Регулярка для поиска кодов

    if match:
        code = match.group(0)
        print(f"Найден код: {code}")
        enter_binance_code(code)

# === Запуск Telegram-клиента ===
client = TelegramClient("session_name", API_ID, API_HASH)

async def main():
    await client.start(PHONE_NUMBER)
    print("Бот запущен и мониторит группу...")
    client.add_event_handler(handle_new_message, events.NewMessage(chats=GROUP_ID))
    await client.run_until_disconnected()

with client:
    client.loop.run_until_complete(main())
