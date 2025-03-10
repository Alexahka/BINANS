import time
import re
import asyncio
from telethon.sync import TelegramClient, events
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from webdriver_manager.chrome import ChromeDriverManager

# === Настройки (заполни перед запуском) ===
API_ID = "ТВОЙ_TELEGRAM_API_ID"
API_HASH = "ТВОЙ_TELEGRAM_API_HASH"
PHONE = "ТВОЙ_НОМЕР"  # Например, +380123456789
GROUP_ID = -100123456789  # ID Telegram-группы
BINANCE_EMAIL = "ТВОЙ_EMAIL"
BINANCE_PASSWORD = "ТВОЙ_ПАРОЛЬ"

# === Настройка Selenium (без GUI) ===
options = webdriver.ChromeOptions()
options.add_argument("--headless")
options.add_argument("--disable-gpu")
options.add_argument("--no-sandbox")

driver = webdriver.Chrome(ChromeDriverManager().install())

# === Функция входа в Binance ===
def login_binance():
    driver.get("https://www.binance.com/ru/my/coupon")
    time.sleep(5)
    
    if "login" in driver.current_url:
        driver.find_element(By.NAME, "email").send_keys(BINANCE_EMAIL)
        driver.find_element(By.NAME, "password").send_keys(BINANCE_PASSWORD)
        driver.find_element(By.NAME, "password").send_keys(Keys.RETURN)
        time.sleep(5)
    print("[+] Вход в Binance выполнен!")

# === Функция ввода кода ===
def enter_binance_code(code):
    driver.get("https://www.binance.com/ru/my/coupon")
    time.sleep(3)

    try:
        input_box = driver.find_element(By.NAME, "redeemCode")
        input_box.send_keys(code)
        input_box.send_keys(Keys.RETURN)
        print(f"[+] Код {code} успешно активирован!")
    except Exception as e:
        print(f"[-] Ошибка ввода кода: {e}")

# === Telegram-бот ===
async def main():
    async with TelegramClient("session", API_ID, API_HASH) as client:
        print("[+] Бот запущен и следит за группой...")

        @client.on(events.NewMessage(chats=GROUP_ID))
        async def handler(event):
            message = event.message.message
            match = re.search(r"[A-Z0-9]{16,}", message)

            if match:
                code = match.group(0)
                print(f"[!] Найден код: {code}")
                enter_binance_code(code)

        await client.run_until_disconnected()

# === Запуск ===
if __name__ == "__main__":
    login_binance()
    asyncio.run(main())
