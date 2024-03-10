################
# Открытие формы с возвратом значения
################
&НаКлиенте
Процедура КомандаОткрытьФормуВыбора(Команда)
	ПараметрыВыбора = Новый Структура;
	ПараметрыВыбора.Вставить("РежимВыбора",Истина);
	ПараметрыВыбора.Вставить("МножественныйВыбор",Истина); ///Если хотим несколько значений
	ОбработкаВыбора = Новый ОписаниеОповещения("ПриЗакрытииФормыВыбора", ЭтаФорма,"ПодборРеализации");
	ОткрытьФорму("Документ.РеализацияТоваровУслуг.ФормаВыбора",ПараметрыВыбора,
	        ЭтаФорма, , , , ОбработкаВыбора);
КонецПроцедуры

&НаКлиенте
Процедура ПриЗакрытииФормыВыбора(Значение, ДопПараметры) Экспорт
   //Дополнительные условия если необходимо
   //Если ДопПараметры = "ПодборРеализации" тогда
    Если Значение = Неопределено Тогда  ///Если ничего не выбрать - вернется пустое значение (Неопределено)
        Возврат;
    КонецЕсли;
    МассивДокументов = Значение ///Если Множественный Выбор - то вернется массив 
    //КонецЕсли;
КонецПроцедуры

ОповеститьОВыборе(<ЗначениеВыбора>)
################
# Прикрепление картинки
################
 &НаКлиенте
Процедура ВыбратьФайлКартинки(Команда)
    //Создаем оповещение,   именно  процедура  "ОбработатьВыборФайла"  будет вызвана при закрытии окна выбора файла
    Оповещение  =  Новый ОписаниеОповещения("ОбработатьВыборФайла",   ЭтотОбъект);
    //Открываем интерактивно  окно для выбора файла
    НачатьПомещениеФайла(Оповещение,   ,   ,   Истина,   УникальныйИдентификатор);
КонецПроцедуры   

&НаКлиенте
Процедура ОбработатьВыборФайла(Результат, Адрес, ВыбранноеИмяФайла, ДополнительныеПараметры) Экспорт
    Если Не Результат Тогда
        Возврат; 
    КонецЕсли;
    СсылкаНаКартинку = Адрес;
КонецПроцедуры

&НаСервере
Процедура Перед3аписьюНаСервере(Отказ, ТекущийОбьект, ПараметрыЗаписи)
    Если ЭтоАдресВременногоХранилища(СсылкаНаКартинку) Тогда
        ТекущийОбьект.ДанныеКартинки = Новый ХранилищеЗначения( ПолучитьИзВремеиногоХранилища(СсылкаНаКартинку) );
    КонецЕсли; 
КонецПроцедуры

&НаСервере
Процедура ПриСозданииНаСервере(Отказ, СтандартнаяОбработка)
    СсылкаНаКартинку = ПолучитьНавигационнуюСсылку(Объект.Ссылка, "ДанныеКартинки"); 
КонецПроцедуры
################
# Чтение json
################
Функция ПростоеЧтениеJSON(Данные)
	ЧтениеJSON = Новый ЧтениеJSON;
	ЧтениеJSON.УстановитьСтроку(Данные);  		
	Возврат ПрочитатьJSON(ЧтениеJSON);
КонецФункции
################
# Чтение excel
################
&НаСервере
Процедура ПрочитатьExcel_ТабличныйДокумент(ПутьКФайлу)
	ТаблицаТоваров.Очистить();
	ТабДок = Новый ТабличныйДокумент;
	Попытка
		ТабДок.Прочитать(ПутьКФайлу);
	Исключение
		Сообщение = Новый СообщениеПользователю;
		Сообщение.Текст = "Не удалось прочитать файл по причине: " + ОписаниеОшибки();
		Сообщение.Сообщить();                                                        
		Возврат;
	КонецПопытки;
	КоличествоСтрок = ТабДок.ВысотаТаблицы;
	КоличествоКолонок = ТабДок.ШиринаТаблицы;
	Для НомерСтроки = 1 По КоличествоСтрок Цикл
		Если НомерСтроки = 1 Тогда
			Продолжить;			
		КонецЕсли;
		СтрокаТаблицы = ТаблицаТоваров.Добавить();
		СтрокаТаблицы.Наименование	= ТабДок.Область(НомерСтроки, 1).Текст;
		СтрокаТаблицы.Артикул		= ТабДок.Область(НомерСтроки, 2).Текст;
		СтрокаТаблицы.Остаток		= ТабДок.Область(НомерСтроки, 3).Текст;
		СтрокаТаблицы.Цена			= ТабДок.Область(НомерСтроки, 4).Текст;
	КонецЦикла;
КонецПроцедуры
################
# Чтение файлов
################
Текст = Новый ЧтениеТекста;
Текст.Открыть("C:\Док.txt");              
Строка = Текст.ПрочитатьСтроку();
Пока Строка <> Неопределено Цикл //строки читаются до символа перевода строки                  
    Позиция = Найти(Строка, "=");
    Если Позиция > 0 Тогда
        Значение = СокрЛП(Сред(Строка, Позиция + 1));
        Сообщить(Значение);
    КонецЕсли;
    Строка = Текст.ПрочитатьСтроку();             
КонецЦикла;

ДиалогВыбора = Новый ДиалогВыбораФайла(РежимДиалогаВыбораФайла.Открытие);
ДиалогВыбора.Заголовок = "Выберите файл";
Если ДиалогВыбора.Выбрать() Тогда
    ИмяФайла = ДиалогВыбора.ПолноеИмяФайла; //Выбор файла из диалогового окна
КонецЕсли;
Текст = Новый ЧтениеТекста(ИмяФайла);
Строка = Текст.ПрочитатьСтроку();
Пока Строка <> Неопределено Цикл //строки читаются до символа перевода строки
    // манипуляции со строкой
    Сообщить(Строка);
    Строка = Текст.ПрочитатьСтроку();
КонецЦикла;
