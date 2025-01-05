# YashasTestTask
Yasha's Test Task

package main

import (
	"fmt"
	"log"
	"strconv"

	tgbotapi "github.com/go-telegram-bot-api/telegram-bot-api/v5"
)

var F = 0

func main() {

	bot, err := tgbotapi.NewBotAPI("7956957657:AAGO4ha8ONjsAeWbQx8QP4qJj5uJq-qhp08")
	if err != nil {
		log.Panic(err)
	}

	bot.Debug = true

	u := tgbotapi.NewUpdate(0)
	u.Timeout = 60
	updates := bot.GetUpdatesChan(u)

	for update := range updates {
		if update.Message != nil {
			if update.Message.IsCommand() {
				handleCommands(bot, update.Message)
			} else {
				handleTextMessages(bot, update.Message)
			}
		} else if update.CallbackQuery != nil {
			handleCallbacks(bot, update.CallbackQuery)
		}
	}
}

func handleCommands(bot *tgbotapi.BotAPI, msg *tgbotapi.Message) {
	switch msg.Command() {
	case "start":
		sendWelcomeMessage(bot, msg.Chat.ID)
	}
}

func sendWelcomeMessage(bot *tgbotapi.BotAPI, chatID int64) {
	msg := tgbotapi.NewMessage(chatID, "Привет! Меня зовут GiftCardBot. Я умею покупать Gift Cards по серийному номеру или по стране, показывать актуальные карты на аккаунте и показать текущий баланс или задолженность.")
	keyboard := tgbotapi.NewInlineKeyboardMarkup(tgbotapi.NewInlineKeyboardRow(tgbotapi.NewInlineKeyboardButtonData("Использовать GiftCardBot", "request_use")))
	msg.ReplyMarkup = keyboard
	bot.Send(msg)
}

func handleTextMessages(bot *tgbotapi.BotAPI, msg *tgbotapi.Message) {
	chatID := msg.Chat.ID
	input := msg.Text

	switch F {
	case 1:
		if len(input) != 6 {
			bot.Send(tgbotapi.NewMessage(chatID, "Номер должен состоять из 6 цифр. Пожалуйста, введите ещё раз."))
		} else {
			cards := 5000 //SELECT id, country FROM ...
			if cards == 0 {
				bot.Send(tgbotapi.NewMessage(chatID, "Совпадений нет. Пожалуйста, введите номер ещё раз."))
			} else {
				bot.Send(tgbotapi.NewMessage(chatID, fmt.Sprintf("Есть %d gift cards с первыми 6 цифрами номера (%s)", cards, input)))
				bot.Send(tgbotapi.NewMessage(chatID, "Файл с вашими картами:"))
			}
			F = 0
		}
	case 2:
		if len(input) != 2 {
			bot.Send(tgbotapi.NewMessage(chatID, "Страна должна быть указана двумя буквами. Пожалуйста, введите ещё раз."))
		} else {
			cards := 5000 //SELECT id, country FROM ...
			if cards == 0 {
				bot.Send(tgbotapi.NewMessage(chatID, "Совпадений нет. Пожалуйста, введите страну ещё раз."))
			} else {
				bot.Send(tgbotapi.NewMessage(chatID, fmt.Sprintf("Есть %d gift cards с первыми 6 цифрами номера (%s)", cards, input)))
				bot.Send(tgbotapi.NewMessage(chatID, "Файл с вашими подарочными картами:"))
			}
			F = 0
		}
	case 3:
		if len(input) != 6 {
			bot.Send(tgbotapi.NewMessage(chatID, "Номер должен состоять из 6 цифр. Пожалуйста, введите ещё раз."))
		} else {
			cards := 5000 //SELECT id, country FROM ...
			if cards == 0 {
				bot.Send(tgbotapi.NewMessage(chatID, "Совпадений нет. Пожалуйста, введите страну ещё раз."))
			} else {
				bot.Send(tgbotapi.NewMessage(chatID, fmt.Sprintf("Доступно %d gift cards со страной (%s). Сколько gift cards вы желаете приобрести?", cards, input)))
			}
			F = 5
		}
	case 4:
		if len(input) != 2 {
			bot.Send(tgbotapi.NewMessage(chatID, "Страна должна быть указана двумя буквами. Пожалуйста, введите ещё раз."))
		} else {
			cards := 5000 //SELECT id, country FROM ...
			if cards == 0 {
				bot.Send(tgbotapi.NewMessage(chatID, "Совпадений нет. Пожалуйста, введите страну ещё раз."))
			} else {
				bot.Send(tgbotapi.NewMessage(chatID, fmt.Sprintf("Доступно %d gift cards со страной (%s). Сколько gift cards вы желаете приобрести?", cards, input)))
			}
			F = 5
		}
	case 5:
		quantityInt, err := strconv.Atoi(input)
		if err != nil || quantityInt <= 0 {
			msg := tgbotapi.NewMessage(chatID, "Пожалуйста, введите корректное количество.")
			bot.Send(msg)
			return
		}
		// Здесь должна быть логика для генерации файла и отправки
		msg := tgbotapi.NewMessage(chatID, "Файл с вашими картами:")
		bot.Send(msg)
	default:
		response := tgbotapi.NewMessage(chatID, "Я не умею свободно читать сообщения, нажми на одну из кнопок и я смогу тебя понять ;)")
		bot.Send(response)
		MainMenu(bot, chatID)
	}
}

func handleCallbacks(bot *tgbotapi.BotAPI, query *tgbotapi.CallbackQuery) {
	switch query.Data {
	case "request_use":
		MainMenu(bot, query.Message.Chat.ID)
	case "check_balance":
		checkBalance(bot, query.Message.Chat.ID)
	case "list_by_number":
		listByNumber(bot, query.Message.Chat.ID)
	case "list_by_country":
		listByCountry(bot, query.Message.Chat.ID)
	case "buy_by_number":
		buyByNumber(bot, query.Message.Chat.ID)
	case "buy_by_country":
		buyByCountry(bot, query.Message.Chat.ID)
	}
}

func MainMenu(bot *tgbotapi.BotAPI, chatID int64) {
	keyboard := tgbotapi.NewInlineKeyboardMarkup(
		tgbotapi.NewInlineKeyboardRow(
			tgbotapi.NewInlineKeyboardButtonData("Мои Gift Cards по номеру", "list_by_number"),
			tgbotapi.NewInlineKeyboardButtonData("Мои Gift Cards по стране", "list_by_country")),
		tgbotapi.NewInlineKeyboardRow(
			tgbotapi.NewInlineKeyboardButtonData("Покупка Gift Cards по номеру", "buy_by_number"),
			tgbotapi.NewInlineKeyboardButtonData("Покупка Gift Cards по стране", "buy_by_country")),
		tgbotapi.NewInlineKeyboardRow(
			tgbotapi.NewInlineKeyboardButtonData("Мой баланс", "check_balance")))
	msg := tgbotapi.NewMessage(chatID, "Функции бота:")
	msg.ReplyMarkup = keyboard
	bot.Send(msg)
}

func checkBalance(bot *tgbotapi.BotAPI, chatID int64) {
	msg := tgbotapi.NewMessage(chatID, "Ваша задолженность: 4000")
	bot.Send(msg)
}

func listByNumber(bot *tgbotapi.BotAPI, chatID int64) {
	msg := tgbotapi.NewMessage(chatID, "Введите номер подарочной карты (Первые 6 цифр):")
	bot.Send(msg)
	F = 1
}

func listByCountry(bot *tgbotapi.BotAPI, chatID int64) {
	msg := tgbotapi.NewMessage(chatID, "Введите название подарочной карты (RU/CA/IL):")
	bot.Send(msg)
	F = 2
}

func buyByNumber(bot *tgbotapi.BotAPI, chatID int64) {
	msg := tgbotapi.NewMessage(chatID, "Введите номер подарочной карты, которую жедаете приобрести (Первые 6 цифр):")
	bot.Send(msg)
	F = 3
}

func buyByCountry(bot *tgbotapi.BotAPI, chatID int64) {
	msg := tgbotapi.NewMessage(chatID, "Введите название страны подарочной карты, которую жедаете приобрести (RU/CA/IL):")
	bot.Send(msg)
	F = 4
}
