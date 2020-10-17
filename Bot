import telebot
import requests
import time
import threading
import datetime
import pytz

needed_times = [1603071000]
working = False
not_killed = True


class HackBashAutoLiga:
    tg_token = "1211558301:AAGyrP7Vtm2X4cVdLfnXfZ8b6jGOGvJuNi8"
    phone = "79174738333"
    password = "hWRnTs"
    device_token = "fbhD8ogTQBmdRSz2FK565_:APA91bEWAewOJ_WN7KgQt5NNvxSc3kG8uqMvK-qw7tza3ctI4QFciB6jYLd1kkYG4CTeLRTzYuju7Hqzvn9QmZyE-WNx24LLX-6L3H-sxDv6d8SYR9mXObETQVsvWiQP4Roc8LZm5p6N"
    base_url = "https://api.bashavtoliga.ru"
    secret = "U8Tr6zj7AiNqh4qzRyAB3nBWZpXqPfPCwSHY9J4fkhwTxTPipFRXWJRNycuNW7eaB2iSpbX4Rwc3YnFAXU4XJFhU2QmfQ6qW8hMc"

    chat_ids = []
    allowed_users = ["snitron", ""]

    def __init__(self):
        self.bot = telebot.TeleBot(self.tg_token)

        r = requests.post(self.base_url + "/login",
                          data={"mobile_token": self.device_token, "phone": self.phone, "password": self.password,
                                "secret": self.secret})
        self.access_token = r.json()['access_token']

        self.send_message("Авторизация прошла успешно. Токен: {0}".format(self.access_token))

        @self.bot.message_handler(commands=["start"])
        def start_message(message):
            if message.from_user.username in self.allowed_users:
                self.chat_ids.append(message.chat.id)
                self.bot.reply_to(message, "Привет, кожанный ублюдок!")

        @self.bot.message_handler(commands=["set"])
        def set_needed_times(message):
            text = str(message.text).replace("/set ", "")
            global needed_times
            needed_times.extend(map(int, text.split(" ")))
            self.bot.reply_to(message, "Времена добавлены в мониторинг")

        @self.bot.message_handler(commands=["list"])
        def list_times(message):
            response = ""
            global needed_times
            for i in range(len(needed_times)):
                response += "{0}.  ".format(i) + datetime.datetime.fromtimestamp(
                    needed_times[i]).ctime() + " ({0})".format(needed_times[i]) + "\n"

            self.bot.reply_to(message, response)

        @self.bot.message_handler(commands=["delete"])
        def delete_time(message):
            try:
                text = str(message.text).replace("/delete ", "")
                index = int(text)

                global needed_times

                if index in range(len(needed_times) + 1):
                    needed_times.remove(needed_times[index])
                    list_times(message)
                else:
                    raise Exception()
            except Exception as e:
                self.bot.reply_to(message, "Некорректный индекс элемента (/list)")

        @self.bot.message_handler(commands=["start_monitoring"])
        def start_monitoring(message):
            global working
            working = True

            th = threading.Thread(target=self.get_schedules_and_submit())
            th.start()
            self.bot.reply_to(message, "Мониторинг начат")

        @self.bot.message_handler(commands=["stop_monitoring"])
        def stop_monitoring(message):
            global working
            working = False
            self.bot.reply_to(message, "Мониторинг остановлен")

        self.bot.polling()

    def get_schedules_and_submit(self):
        while working:
            ttime = datetime.datetime.now(pytz.timezone("Asia/Yekaterinburg")).timestamp()
            print(ttime)
            r = requests.get(self.base_url + "/schedules?time={0}".format(round(ttime + 86400)),
                             headers={"Authorization": "Bearer {0}".format(self.access_token)})
            entities = r.json()

            self.send_message("Открытых дат: {0}".format(self.open_count(list(entities))))

            global needed_times
            for i in entities:
                if i['datetime'] in needed_times:
                    if i['status'] == 0 or i['status'] == 1:
                        submit = requests.patch(self.base_url + "/schedules/{0}".format(i['id']),
                                                headers={"Authorization": "Bearer {0}".format(self.access_token)})
                        code = submit.json()['status']
                        self.send_message("Заявка на время {0} отправлена и имеет статус {1}".format(
                            datetime.datetime.fromtimestamp(i['datetime']).ctime(), code))
            time.sleep(2)

    def open_count(self, ll):
        count = 0

        ll = list(ll)
        for i in ll:
            if i['status'] == 0 or i['status'] == 1:
                count += 1

        return count

    def send_message(self, message):
        for i in self.chat_ids:
            self.bot.send_message(i, message)


hacker = HackBashAutoLiga()
