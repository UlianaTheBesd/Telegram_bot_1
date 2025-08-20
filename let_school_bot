import telebot
from telebot import types
import logging
from os import getenv

# ЛОГИРОВАНИЕ (необязательно).
logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO
)

# ТОКЕН БОТА.
BOT_TOKEN = getenv("BOT_TOKEN", "(there was my bot's id, I made it here: @BotFather)")

# ТОКЕН АДМИНИСТРАТОРА (пересылка информации от клиента).
ADMIN_CHAT_ID = getenv("ADMIN_CHAT_ID", "(there was an id, I made it in this bot: @userinfobot)")


bot = telebot.TeleBot(BOT_TOKEN)

user_data = {} # Словарь для хранения данных пользователя.

#                                                               Начало работы.
@bot.message_handler(commands=['start'])
def start_message(message):
    user = message.from_user

    if str(user.id) == ADMIN_CHAT_ID:
        admin_welcome_message = (
            f"Приветствую вас, {user.full_name}!\n"
            "Вы вошли в режим ожидания сообщений от пользователей.\n"
            "При регистрации заявка нового клиента будет отображаться в этом чате."
        )
        bot.send_message(message.chat.id, admin_welcome_message)
        return

    welcome_message = (
        "Мы рады приветствовать вас в школе иностранных языков \"LET\"!\n"
        "Записаться к нам на занятие вы можете заполнив эту форму.\n"
        "Мы с вами обязательно свяжемся."
    )

    markup = types.ReplyKeyboardMarkup(one_time_keyboard=True, resize_keyboard=True)
    markup.add(types.KeyboardButton("Начать"))

    bot.send_message(
        message.chat.id,
        f"Здравствуйте, {user.full_name}!\n\n{welcome_message}"
    )
    
    user_data[message.chat.id] = {
        'telegram_user': user.full_name,
        'child_name': None,
        'parent_phone': None,
        'school_info': None,
        'school_shift': None,
        'study_year': None,
        'english_grade': None,
    }

    bot.send_message(message.chat.id, "Для записи, пожалуйста, введите фамилию и имя ребёнка.\n")

    bot.register_next_step_handler(message, get_child_name)

def show_summary(message):
    chat_id = message.chat.id
    if chat_id not in user_data:
        bot.send_message(chat_id, "Пожалуйста, начните заполнение формы, написав /start.")
        return

    data = user_data[chat_id]
    summary_text = (
        "**Всё ли указано верно?**\n\n"
        f"1. Фамилия и имя ребёнка: `{data['child_name']}`\n"
        f"2. Номер телефона родителя: `{data['parent_phone']}`\n"
        f"3. Номер школы и класс: `{data['school_info']}`\n"
        f"4. Смена в школе: `{data['school_shift']}`\n"
        f"5. Год обучения в LET: `{data['study_year']}`\n"
        f"6. Отметка по английскому: `{data['english_grade']}`\n\n"
    )
    
    markup = types.ReplyKeyboardMarkup(one_time_keyboard=True, resize_keyboard=True)
    markup.add(types.KeyboardButton("Всё верно"))
    markup.add(types.KeyboardButton("Изменить"))

    bot.send_message(
        chat_id,
        summary_text,
        parse_mode='Markdown',
        reply_markup=markup
    )
    bot.register_next_step_handler(message, handle_summary_choice)


#                                                              Поля для ввода.
def get_child_name(message):
    chat_id = message.chat.id
    user_data[chat_id]['child_name'] = message.text
    bot.send_message(chat_id, "Введите номер телефона родителя в формате +79XXXXXXXXX.")
    bot.register_next_step_handler(message, get_parent_phone)

def get_parent_phone(message):
    chat_id = message.chat.id
    phone_number = message.text
    if phone_number.startswith('+7') and phone_number[1:].isdigit() and len(phone_number) == 12:
        user_data[chat_id]['parent_phone'] = phone_number
        bot.send_message(chat_id, "Пожалуйста, укажите номер школы и класс с буквой (например: 44 5А).")
        bot.register_next_step_handler(message, get_school_info)
    else:
        bot.send_message(chat_id, "Кажется, номер телефона введён неверно. Пожалуйста, попробуйте ещё раз в формате +79XXXXXXXXX.")
        bot.register_next_step_handler(message, get_parent_phone)

def get_school_info(message):
    chat_id = message.chat.id
    user_data[chat_id]['school_info'] = message.text
    markup = types.ReplyKeyboardMarkup(one_time_keyboard=True, resize_keyboard=True)
    markup.add(types.KeyboardButton("Первая"), types.KeyboardButton("Вторая"), types.KeyboardButton("Пока не знаю"))
    bot.send_message(chat_id, "Какая смена в школе?", reply_markup=markup)
    bot.register_next_step_handler(message, get_school_shift)

def get_school_shift(message):
    chat_id = message.chat.id
    user_data[chat_id]['school_shift'] = message.text
    bot.send_message(chat_id, "Какой год обучения в нашей школе LET?", reply_markup=types.ReplyKeyboardRemove())
    bot.register_next_step_handler(message, get_study_year)

def get_study_year(message):
    chat_id = message.chat.id
    user_data[chat_id]['study_year'] = message.text
    bot.send_message(chat_id, "Какая отметка по английскому языку в вашей школе?")
    bot.register_next_step_handler(message, get_english_grade)

def get_english_grade(message):
    chat_id = message.chat.id
    user_data[chat_id]['english_grade'] = message.text
    show_summary(message)


#                                                              "Всё ли верно?". 
def handle_summary_choice(message):
    chat_id = message.chat.id
    if message.text == "Всё верно":
        data = user_data[chat_id]
        admin_message = (
            f"**Новая заявка на запись**\n\n"
            f"От пользователя Telegram: {data.get('telegram_user')}\n\n"
            f"1. Фамилия и имя ребёнка: `{data.get('child_name')}`\n"
            f"2. Номер телефона родителя: `{data.get('parent_phone')}`\n"
            f"3. Номер школы и класс: `{data.get('school_info')}`\n"
            f"4. Смена в школе: `{data.get('school_shift')}`\n"
            f"5. Год обучения в LET: `{data.get('study_year')}`\n"
            f"6. Отметка по английскому: `{data.get('english_grade')}`"
        )
        try:
            bot.send_message(chat_id=ADMIN_CHAT_ID, text=admin_message, parse_mode='Markdown')
            bot.send_message(chat_id, "Спасибо за выбор нашей школы. Наш администратор обязательно с вами свяжется.", reply_markup=types.ReplyKeyboardRemove())
        except Exception as e:
            bot.send_message(chat_id, "Произошла ошибка при отправке данных. Пожалуйста, попробуйте начать заново, написав /start.", reply_markup=types.ReplyKeyboardRemove())
            logging.error(f"Failed to send message to admin: {e}")

        user_data.pop(chat_id)
    
    elif message.text == "Изменить":
        bot.send_message(chat_id, "Пожалуйста, введите номер вопроса, который хотите изменить (1-6).", reply_markup=types.ReplyKeyboardRemove())
        bot.register_next_step_handler(message, handle_update_choice)
    else:
        bot.send_message(chat_id, "Пожалуйста, выберите 'Всё верно' или 'Изменить'.")
        bot.register_next_step_handler(message, handle_summary_choice)


def handle_update_choice(message):
    chat_id = message.chat.id
    try:
        question_number = int(message.text)
        if 1 <= question_number <= 6:
            user_data[chat_id]['update_state'] = question_number 
            bot.send_message(chat_id, f"Хорошо, введите новый ответ на вопрос #{question_number}.")
            bot.register_next_step_handler(message, handle_update_response)
        else:
            bot.send_message(chat_id, "Пожалуйста, введите число от 1 до 6.")
            bot.register_next_step_handler(message, handle_update_choice)
    except ValueError:
        bot.send_message(chat_id, "Это не похоже на число. Пожалуйста, введите число от 1 до 6.")
        bot.register_next_step_handler(message, handle_update_choice)

def handle_update_response(message):
    chat_id = message.chat.id
    state_to_update = user_data[chat_id].get('update_state')
    
    question_map = {
        1: 'child_name',
        2: 'parent_phone',
        3: 'school_info',
        4: 'school_shift',
        5: 'study_year',
        6: 'english_grade',
    }

    key_to_update = question_map.get(state_to_update)

    if key_to_update:
        if key_to_update == 'parent_phone':
            phone_number = message.text
            if phone_number.startswith('+7') and phone_number[1:].isdigit() and len(phone_number) == 12:
                user_data[chat_id][key_to_update] = phone_number
            else:
                bot.send_message(chat_id, "Кажется, номер телефона введён неверно. Пожалуйста, попробуйте ещё раз в формате +79XXXXXXXXX.")
                bot.register_next_step_handler(message, handle_update_response)
                return
        else:
            user_data[chat_id][key_to_update] = message.text

    user_data[chat_id].pop('update_state', None)
    
    show_summary(message)

def main() -> None:
    bot.polling(non_stop=True)

if __name__ == "__main__":
    main()
