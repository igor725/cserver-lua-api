# Getting started
Итак, вы решили, что написать Lua скрипт для вас - это слишком легко. Поэтому ища трудности себе на пятую точку, вами было принято решение написать нативный плагин для сервера. Для того чтобы эту затею было проще осуществить, ниже приведён минимальный пример плагина для сервера.

```c
#include <core.h> // Основной набор вспомогательных типов и дефайнов, подключается каждым хедером сервера
#include <plugin.h> // Дефайны и функции, необходимые для создания плагина
#include <event.h> // Структуры, функции и прочие непотребства для взаимодействия с системой эвентов

Plugin_SetVersion(1); // Вызов этого макроса обязателен - производится установка не только версии плагина, но и используемой версии PluginAPI
Plugin_SetURL("https://docs.igvx.ru/"); // Домашняя страница плагина, указывается для получения фидбека

static void evtpoststart(void) {
	// Тут должны происходить какие-то важные дела
}

static void evtontick(cs_int32 *delta) {
	// Тут тоже желательно выполнять какие-нибудь важные дела, но не сильно тяжёлые, а то сервер обидится
}

Event_DeclareBunch (events) { // Определяем массив "events" содержащий список наших эвент-коллбеков
	// Буква 'v' первым аргументом указвает на то, что функция evtpoststart ничего не возвращает, некоторые эвенты поддерживают обработку возвращённых значений (см. документацию по эвентам), для определения эвента, который возвращает boolean значения заменить 'v' на 'b'
	EVENT_BUNCH_ADD('v', EVT_POSTSTART, evtpoststart), // Данный эвент выполнится лишь единожды - при запуске сервера
	EVENT_BUNCH_ADD('v', EVT_ONTICK, evtontick), // Эта функция вызывается при каждом тике сервера

	EVENT_BUNCH_END // Каждый список эвентов должен кончаться данным макросом
};

/* В нашем случае функция Plugin_Load пытается зарегистрировать эвенты, указанные в массиве выше,
** если у неё это не получится, то плагин выгрузится. А функция Plugin_Unload отрегистрирует все
** эвенты в массиве. Важно: если не производить с умом освобождение ресурсов, то сервер может
** обвалиться в самый неподходящий момент! */

cs_bool Plugin_Load(void) {
	return Event_RegisterBunch(events); // Если эта функция вернёт false, плагин будет немедленно выгружен (вызов функции Plugin_Unload будет осуществлён), а его библиотека отключена
}

cs_bool Plugin_Unload(cs_bool force) {
	return Event_UnregisterBunch(events); // Если эта функция вернёт false, то выгрузка плагина будет отменена, но если параметр force установлен в true, то возвращаемое значение будет проигнорировано
}
```

Полный список всевозможных событий вы можете найти в навигационном меню слева.